[TOC]

# 七、Pixel Buffer






# 八、Frame Buffer

> 定义：framebuffer 是 OpenGL 一系列数据存储的集合

## 1. 不同种类的 framebuffer

1. **Default framebuffer**
   本地窗口系统创建和使用的 framebuffer，一定会显示到屏幕上，是本地系统创建 Context 的一部分
   这种 framebuffer 有本地窗口系统 API 创建提供，由 OpenGL 将其作为自己的输出给本地窗口系统来使用
   包含：多个（至少一个）色彩缓冲、一个深度缓冲、一个模板缓冲、一个累积缓冲

  2. **Frame Buffer Object**
     OpenGL 创建和使用的 framebuffer，可以不显示到屏幕上
     提供一个 FBO 对象来供 OpenGL 操作，FBO 对象可以有**多个（至少一个）色彩缓冲**，一个深度缓冲，一个模板缓冲（没有累积缓冲）



## 2. framebuffer 内部数据的切换

framebuffer 提供的内部数据切换方式都要比单独切换 framebuffer 自己要快

切换 texture：**glFramebufferTexture2D**
切换 renderbuffer：**glFramebufferRenderbuffer**




## 3. renderbuffer





## 4. 检查 framebuffer 的状态



# 引用

1. [OpenGL Pixel Buffer Object](http://www.songho.ca/opengl/gl_pbo.html)
2. [OpenGL Frame Buffer Object](http://www.songho.ca/opengl/gl_fbo.html)