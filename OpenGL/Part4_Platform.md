[TOC]

# 零、代码 Debug

> 关于性能的衡量 帧率/单帧时间，**建议采用单帧时间**
>
> 由于 帧率 = 60 / 单帧时间，他们之间的关系不是线性的，例如在减少同样时间消耗的情况下：
> 低帧率区间进步缓慢，每秒 10fps 下，帧时间减少 2ms，帧率提高到 10.2fps
> 高帧率区间进步明显，每秒 25fps 下，帧时间减少 2ms，帧率提高到 26.3fps



# 一、Android 平台

>图片有不清晰的地方，可以点开连接看大图

## 1. 数据的封装

### 1.1 Surface
内存中的一段绘图缓冲区，对 framebuffer 的 Java 封装对象



### 1.2 SurfaceTexture
内存中的一段绘图缓冲区，对 EGL texture 的 Java 封装对象

数据源：android.hardware.camera2, MediaCodec, MediaPlayer 和 Allocation 这些类的目标视频数据输出对象

限制：API 11 存在，Android 3.0 及其后才能使用

特点：可以接收一个 EGL texture 的纹理 ID 来产生，可以做到**离线渲染**

关键方法：updateTexImage（从内容流中获取当前帧，使得内容流中的一些帧可以跳过）

![](./images/surfaceTexture.png)



SurfaceTexture 使用流程

![](./images/processSurfaceTexture.png)



## 2. 数据的展示和刷新

### 2.1 SurfaceView

父类：View

限制：API 1 存在

回调方法运行线程：主线程

优点：有自己的 Surface 来刷新，可以做到**局部刷新，独立线程刷新数据**

缺点：不能进行 Transition，Rotation，Scale 等变换，这导致 SurfaceView 在由于刷新不受主线程控制，**滑动时可能有黑边**

关键方法：getHolder（获取 SurfaceHolder ，SurfaceHolder 持有 Surface 数据对象）



### 2.2 GLSurfaceView

父类：SurfaceView

限制：API 3 存在，Android 1.5 及其后才能使用

关键方法：

![](./images/glsurfaceview.png)



### 2.3 TextureView

父类：View

限制：API 14 存在，Android 4.0 及其后才能使用，只能针对用于硬件加速（没有 GPU 就无法使用）

回调方法运行线程：主线程（在Android 5.0 引入渲染线程后，它是在渲染线程中做的）

特点：没有自己的 Surface 来刷新，使用所在的 window 来**全局刷新**，可以进行 Transition，Rotation，Scale 等变换，**滑动时没有黑边**

关键方法：

- getSurfaceTexture（可能返回 null）

- setSurfaceTextureListener

  ```java
  public DrawTextureView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    this.setSurfaceTextureListener(this);
  }
  
  @Override
  public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
    mSurface = new Surface(surface);
  }
  
  @Override
  public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {}
  
  @Override
  public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {  
    return true;
  }
  
  @Override
  public void onSurfaceTextureUpdated(SurfaceTexture surface) {}
  ```
  ![](./images/textureView.jpeg)




### 2.4 Android 5.0 引入渲染线程

![](./images/renderThread.jpeg)



## 3. EGL 环境配置

> 如果 OpenGL 是打印机，EGL 就是纸。EGL：作为 OpenGL ES 和本地窗口的桥梁，不同平台的 EGL 实现方式不一样


### 3.1 渲染同步问题


本地环境和客户端环境

- 本地窗口环境：Windows、X
- 客户端环境：3D 渲染器 OpenGL、2D 矢量图形渲染器 [OpenVG](https://baike.baidu.com/item/OpenVG/7922699?fr=aladdin)



同一个 surface 上可能 **同时异步** 执行了 <u>本地窗口环境</u> 和 <u>客户端环境</u> 的命令

- 本地环境等待客户端环境渲染完成，效果同 glFinish 或 vgFinish 一致
  [EGLBoolean eglWaitClient(void);](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglWaitClient.xhtml)
- 客户端环境等待本地环境渲染完成
  [EGLBoolean eglWaitNative(EGLint engine);](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglWaitNative.xhtml)



### 3.2 获取 EGL 版本信息

1. [EGLBoolean eglInitialize(EGLDisplay dpy, EGLint *major, EGLint *minor);](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglInitialize.xhtml)
   本质是初始化函数，但也能通过返回 major，minor 来获取当前 display 硬件设备支持的 OpenGL ES 版本号
2. [const char* eglQueryString(EGLDisplay dpy, EGLint name);](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglQueryString.xhtml)
   查询当前 display 硬件设备有哪些  EGL 的扩展支持，**调用前必须先通过 eglInitialize 初始化 EGL**



### 3.3 EGL 的 Context 和 Surface

EGL 可以销毁本地资源（各种 surface 类型）
只有一个方法，[EGLBoolean eglDestroySurface(EGLDisplay display, EGLSurface surface);](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglDestroySurface.xhtml)

EGL 可以创建本地环境的资源（各种 surface 类型）格式有

1. pixel buffer（存储在显存上）
   OpenGL API 方面，[EGLSurface eglCreatePbufferSurface(...)](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglCreatePbufferSurface.xhtml)
   OpenVG API 方面，[EGLSurface eglCreatePbufferFromClientBuffer(...)](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglCreatePbufferFromClientBuffer.xhtml)
2. frame buffer（存储在显存上）
   本地环境 API 创建的 window 内的 surface，[EGLSurface eglCreateWindowSurface(...)](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglCreateWindowSurface.xhtml)
   
3. 本地环境 API 创建的 [pixmap（常用做本地图像的内部存储）](https://franz.com/support/documentation/current/doc/cg/cg-pixmaps.htm)，[EGLSurface eglCreatePixmapSurface(...);](https://www.khronos.org/registry/EGL/sdk/docs/man/html/eglCreatePixmapSurface.xhtml)
	Pixmap：将图像以像素颜色数组的结构来存储的对象
	Bitmap：用 1 bit 来存储 1 个像素颜色的 Pixmap



**EGLContext 上下文创建步骤**

```c
#include <stdlib.h>
#include <unistd.h>
#include <EGL/egl.h>
#include <GLES/gl.h>
typedef ... NativeWindowType;
extern NativeWindowType createNativeWindow(void);

// 虽然是一维数组，但还是要采用 id, value, id, value ... 的存储方式
static EGLint const attribute_list[] = {
        EGL_RED_SIZE, 1,
        EGL_GREEN_SIZE, 1,
        EGL_BLUE_SIZE, 1,
        EGL_NONE
};
int main(int argc, char ** argv)
{
        EGLDisplay display;	//EGLDisplay：关联系统物理屏幕，表示显示设备句柄
        EGLConfig config;
        EGLContext context;
        EGLSurface surface;
        NativeWindowType native_window;
        EGLint num_config;

        /* 1. get an EGL display connection */
        display = eglGetDisplay(EGL_DEFAULT_DISPLAY);

        /* 2. initialize the EGL display connection */
        eglInitialize(display, NULL, NULL);

        /* 3. get an appropriate EGL frame buffer configuration */
        eglChooseConfig(display, attribute_list, &config, 1, &num_config);

        /* 4. create an EGL rendering context */
        context = eglCreateContext(display, config, EGL_NO_CONTEXT, NULL);

        /* 5. create a native window */
        native_window = createNativeWindow();

        /* 6. create an EGL window surface */
        surface = eglCreateWindowSurface(display, config, native_window, NULL);

        /* 7. connect the context to the surface */
        eglMakeCurrent(display, surface, surface, context);

        /* 8. clear the color buffer */
        glClearColor(1.0, 1.0, 0.0, 1.0);
        glClear(GL_COLOR_BUFFER_BIT);
        glFlush();

  			// 所有的绘制步骤在后台绘制，当绘制完成时切换前后台缓冲，确保显示的一直是绘制完成的画面
        eglSwapBuffers(display, surface);

        sleep(10);
        return EXIT_SUCCESS;
}
```



## 4. 平台问题

###4.1 TEXTURE_EXTERNAL_OES

[TEXTURE_EXTERNAL_OES](https://www.khronos.org/registry/OpenGL/extensions/OES/OES_EGL_image_external.txt) 是 OpenGL ES 在 Android 上的扩展，在获取相机纹理时只有 TEXTURE_EXTERNAL_OES 类型的纹理

使用扩展纹理 TEXTURE_EXTERNAL_OES 步骤

1. 创建纹理时

   ```java
   // 注意 GLES11Ext
   GLES20.glBindTexture(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, texId);
   
   GLES20.glTexParameterf(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GL11.GL_TEXTURE_MIN_FILTER, GL11.GL_LINEAR);
   GLES20.glTexParameterf(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GL11.GL_TEXTURE_MAG_FILTER, GL11.GL_LINEAR);
   GLES20.glTexParameteri(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GL11.GL_TEXTURE_WRAP_S, GL11.GL_CLAMP_TO_EDGE);
   GLES20.glTexParameteri(GLES11Ext.GL_TEXTURE_EXTERNAL_OES, GL11.GL_TEXTURE_WRAP_T, GL11.GL_CLAMP_TO_EDGE);
   ```

2. 在 fragment shader 里要提前声明使用的扩展

   ```c
   #extension GL_OES_EGL_image_external : require
   uniform samplerExternalOES u_texture; // 代替 sampler2D
   void main() {}
   ```

   

###4.2 Java 成员变量和 C++ 指针的 JNI 层绑定

绑定指针

- Java 类中，声明一个为 long 的长整型类型的成员变量代表 C++ 指针的句柄

- C / C++ 文件中，每次获取 jlong 的成员变量时**强制转换为指针来使用**

  ```c
  #include <jni.h>
  
  jfieldID inline getHandleField(JNIEnv *env, jobject obj)
  {
      jclass c = env->GetObjectClass(obj);
      // J is the type signature for long:
      return env->GetFieldID(c, "nativeHandle", "J");
  }
  
  template <typename T>
  T *getHandle(JNIEnv *env, jobject obj)
  {
      jlong handle = env->GetLongField(obj, getHandleField(env, obj));
      return reinterpret_cast<T *>(handle);
  }
  
  template <typename T>
  void setHandle(JNIEnv *env, jobject obj, T *t)
  {
      jlong handle = reinterpret_cast<jlong>(t);
      env->SetLongField(obj, getHandleField(env, obj), handle);
  }
  
  // Use Example
  MyObject* ptr = new MyObject();
  setHandle<MyObject>(env, object， ptr)；
  ptr = getHandle<MyObject>(env, object);
  ```

  

绑定智能指针

- Java 类中，声明一个为 long 的长整型类型的成员变量代表 C++ 指针的句柄

- C / C++ 文件中，每次获取 jlong 的成员变量时**强制转换为包含智能指针的对象的指针来使用**

  ```c
  template <typename T>
  class SmartPointerWrapper {
      std::shared_ptr<T> mObject;
  public:
      template <typename ...ARGS>
      explicit SmartPointerWrapper(ARGS... a) {
          mObject = std::make_shared<T>(a...);
      }
  
      explicit SmartPointerWrapper (std::shared_ptr<T> obj) {
          mObject = obj;
      }
  
      virtual ~SmartPointerWrapper() noexcept = default;
  
      void instantiate (JNIEnv *env, jobject instance) {
          setHandle<SmartPointerWrapper>(env, instance, this);
      }
  
      jlong instance() const {
          return reinterpret_cast<jlong>(this);
      }
  
      std::shared_ptr<T> get() const {
          return mObject;
      }
  
      static std::shared_ptr<T> object(JNIEnv *env, jobject instance) {
          return get(env, instance)->get();
      }
  
      static SmartPointerWrapper<T> *get(JNIEnv *env, jobject instance) {
          return getHandle<SmartPointerWrapper<T>>(env, instance);
      }
  
      static void dispose(JNIEnv *env, jobject instance) {
          auto obj = get(env,instance);
          delete obj;
          setHandle<SmartPointerWrapper>(env, instance, nullptr);
      }
  };
  
  // Use Example
  SmartPointerWrapper<Object> obj = new SmartPointerWrapper<Object>(arguments);
  obj->instantiate(env,instance);
  ```




## 5. Debug

1. [系统自带的 GPU 呈现分析](https://www.cnblogs.com/ldq2016/p/6667381.html)
2. [高通骁龙 Adreno GPU Profiler 调试工具（建议在 windows 下使用，mac 下测试无用）](https://gameinstitute.qq.com/community/detail/123051)
3. [GAPID 调试 Android 应用，需要 Android stuido 停用 adb 的使用](http://www.geeks3d.com/20171214/google-gapid-capture-vulkan-and-opengl-es-calls-on-android-windows-macos-and-linux/)



# 二、iOS 平台





# 三、QT 平台





# 参考

- [A C++ Smart Pointer wrapper for use with JNI](https://www.studiofuga.com/2017/03/10/a-c-smart-pointer-wrapper-for-use-with-jni/)
- [Android中的 EGL 扩展](http://ju.outofmemory.cn/entry/146313)
- [SurfaceView、SurfaceHolder、Surface](https://blog.csdn.net/holmofy/article/details/66578852)
- [TextureView、SurfaceTexture、Surface](https://blog.csdn.net/Holmofy/article/details/66583879)
- [SurfaceView、TextureView、SurfaceTexture 等的区别](https://www.cnblogs.com/wytiger/p/5693569.html)
- [OpenGL ES：EGL 接口解析与理解](https://blog.csdn.net/xuwei072/article/details/70049004)
- [OpenGL ES：EGL简介](https://blog.csdn.net/iEearth/article/details/71180457)
- [Android中 的 GraphicBuffer 同步机制 Fence](https://blog.csdn.net/jinzhuojun/article/details/39698317)
- [深入 Android 渲染机制](https://www.cnblogs.com/ldq2016/p/6668148.html)
