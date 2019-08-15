[TOC]

# 一、纹理

## 1. 纹理环绕（坐标包装）

> 当**纹理坐标超出默认范围**时，每种纹理环绕方式都有不同的视觉效果输出

OpenGL 设置纹理不同坐标轴的环绕方式

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_NEAREST); //纹理坐标 s/u/x 轴的包装格式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_LINEAR);  //纹理坐标 t/v/y 轴的包装格式
```

![](images/texture_wrapping.png)

| 环绕方式           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| GL_REPEAT          | 对纹理的默认行为，重复纹理图像                               |
| GL_MIRRORED_REPEAT | 和 GL_REPEAT 一样，但每次重复图片是镜像放置的                |
| GL_CLAMP_TO_EDGE   | 纹理坐标会被约束在 0 ～ 1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果 |
| GL_CLAMP_TO_BORDER | 超出的坐标处的纹理为用户指定的边缘颜色                       |



## 2. 纹理过滤（采样）

> 当三维空间里面的多边形，变成二维屏幕上的一组像素的时候，对每个像素需要到相应纹理图像中进行采样，这个过程就称为纹理过滤 

纹理过滤的两种情况

- 纹理被缩小 `GL_TEXTURE_MIN_FILTER`：一个像素对应多个纹理单元
  例，一个 8 X 8 的纹理贴到远处正方形上，最后在屏幕上占了 2 X 2 个像素矩阵
- 纹理被放大 `GL_TEXTURE_MAG_FILTER`：一个纹理单元对应多个像素
  例，一个 2 X 2 的纹理贴到近处正方形上，最后在屏幕上占了 8 X 8 个像素矩阵


OpenGL 中针对放大和缩小的情况的设置

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST); //缩小
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);  //放大
```

### 2.1 最近点采样 GL_NEAREST

优点：效率最高
缺点：效果最差

方法：选择最接近中心点纹理坐标的 **1 个纹理单元**采样

![](images/texture_nearest.png)

### 2.2 双线性过滤 GL_LINEAR

优点：适于处理有一定精深的静态影像
缺点：不适用于绘制动态的物体，当三维物体很小时会产生深度赝样锯齿 (Depth Aliasing artifacts)

方法：选择最接近中心点纹理坐标的 2 X 2 纹理单元矩阵进行采样，取 **4 个纹理单元**采样的平均值

![](images/texture_linear.png)

### 2.3 三线性过滤 GL_LINEAR_MIPMAP_LINEAR

> 多级渐远纹理 (Mipmap) 
> 效率高但是会占用一定的空间
> 一系列的纹理图像，后一个纹理图像是前一个的二分之一。距观察者的距离超过一定的阈值，OpenGL会使用不同的多级渐远纹理，即最适合物体的距离的那个。由于距离远，解析度不高也不会被用户注意到。
>
> 一张方形地板的多级渐远纹理如下图
>
> ![](images/mipmaps.png)

优点：效果最好，适用于动态物体或景深很大的场景
缺点：效率低，只能用于纹理被缩小的情况

方法：

1. 取 Mipmap 纹理中距离与当前屏幕上尺寸相近的两个纹理

2. 将 1 中选取的纹理 选择最接近中心点纹理坐标的 2 X 2 纹理单元矩阵进行采样（线性过滤）

3. 将 2 中两次采样的结果进行加权平均（**8 个纹理单元**采样），得到最后的采样数据


### 2.4 各向异性过滤

> 之前提到的三种过滤方式，默认纹理在 x，y 轴方向上的缩放程度是一致的
> 当纹理在 3D 场景中，出现一个轴的方向纹理放大，一个轴的方向纹理缩小的情况（**OpenGL 判定为纹理缩小**）需要使用各向异性过滤配合以上三种过滤方式来达到最佳的效果

优点：效果最好，使画面更加逼真
缺点：效率最低，由硬件实现

方法：

1. 确定 X、Y 方向的采样比例
   ScaleX = 纹理的宽 / 屏幕上显示的纹理的宽
   ScaleY = 纹理的高 / 屏幕上显示的纹理的高
   异向程度 = max(ScaleX, ScaleY) / min(ScaleX, ScaleY);

   例，64 X 64的纹理最后投影到屏幕上占了128 X 32 的像素矩阵
   ScaleX = 64.0 / 128.0 = 0.5;
   ScaleY = 64.0 / 32.0 = 2.0;
   异向程度 = 2.0 / 0.5 = 4;

2. 根据采样比例分别在 X、Y 方向上采用 *三线性过滤* 或 *双线性过滤* 获得采样数据，**采样的范围由异向程度决定，不是原来的 2 X 2 像素矩阵**

   例，64 X 64的纹理最后投影到屏幕上占了128 X 32 的像素矩阵
   异向程度为 4，且在 缩放方面 X 轴 > Y 轴，所以 X 轴采样 2 个像素，Y 轴采样 2 * 异向程度 = 8 个像素
   采样范围为最接近中心点纹理坐标的 2 X 8 的像素矩阵

OpenGL 中设置各向异性过滤

```c
glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAX_ANISOTROPY_EXT, 异向程度);
```

各向异性对比三线性

![](images/texture_anisotropic.jpg)



# 二、顶点信息传输

## 1. 立即模式 glBegin()/glEnd()

方式：立即绘制

优点：功能适配范围广，写法直观
缺点：频繁调用 OpenGL 函数，效率低，共享点使用次数多

例子：

1. 直接提交 OpenGL 命令到 GPU

```c
// Note that not all of OpenGL commands can be placed in between glBegin() and glEnd()
// Only a subset of commands can be used
// glVertex*(), glColor*(), glNormal*(), glTexCoord*(), glMaterial*(), glCallList(), etc.
glBegin(GL_TRIANGLES);
    glColor3f(1, 0, 0); // set vertex color to red
    glVertex3fv(v1);    // draw a triangle with v1, v2, v3
    glVertex3fv(v2);
    glVertex3fv(v3);
glEnd();
```

2. 将命令放到 DisplayList 后，批量一次传入到 GPU
   DisplayList 会将其中命令的所有资源存储到自己的内存中
   DisplayList 是服务端的状态，本身存储在 GPU 缓存中，只能存储与服务端有关的部分命令
   DisplayList 的命令和数据一旦上传便不可修改

```c
// create one display list
GLuint index = glGenLists(1);

// compile the display list, store a triangle in it
// Option: GL_COMPILE or GL_COMPILE_AND_EXECUTE(render)
glNewList(index, GL_COMPILE);
    glBegin(GL_TRIANGLES);
    glVertex3fv(v0);
    glVertex3fv(v1);
    glVertex3fv(v2);
    glEnd();
glEndList();
...

// draw the display list
glCallList(index);
...

// delete it if it is not used any more
glDeleteLists(index, 1);
```



## 2. VertexArray

方式：批量数据传入绘制

优点：数据以数组的形式**存储在应用缓存**，减少了 OpenGL 函数的频繁调用
缺点：每次绘制都要占用带宽上传到显存

例子：

```c
GLfloat vertices[] = {...}; // 36 of vertex coords

glUseProgram(progId);

// activate and specify pointer to vertex array
// 因为 vertices 存储在应用程序上，所以这里 enable client state
glEnableClientState(GL_VERTEX_ARRAY);
// 也可以用 
// glNormalPointer、glColorPointer、glIndexPointer、glTexCoordPointer、glEdgeFlagPointer
glVertexPointer(3, GL_FLOAT, 0, vertices);

// draw a cube
// 也可以用 glDrawElements、glDrawRangeElements
glDrawArrays(GL_TRIANGLES, 0, 36);

// deactivate vertex arrays after drawing
glDisableClientState(GL_VERTEX_ARRAY);
```



## 3. VertexBuffer

方式：批量数据传入绘制

优点：
1. 数据以数组的形式**存储在显卡高速缓存**，每次使用时不用重新上传，只需要在显卡绑定即可
2. 数据由于存储在显存，可以被应用程序在不同线程访问和修改

例子：

1. 创建和销毁

```c
GLuint vboId;                              // ID of VBO
GLfloat* vertices = new GLfloat[vCount*3]; // create vertex array

// generate a new VBO and get the associated ID
glGenBuffers(1, &vboId);

// bind VBO in order to use
// Option: GL_ARRAY_BUFFER or GL_ELEMENT_ARRAY_BUFFER
// This Option assists VBO to decide the most efficient locations of buffer objects
// For example, some systems may prefer indices in AGP or system memory, and vertices in video memory
// Once glBindBuffer() is first called, VBO initializes the buffer with a zero-sized memory buffer and set the initial VBO states, such as usage and access properties.
glBindBuffer(GL_ARRAY_BUFFER, vboId);

// upload data to VBO
// Option: glBufferSubData, GL_[STATIC/DYNAMIC/STREAM]_[DRAW/READ/COPY]
// GL_STATIC_DRAW 决定了数据的存储位置
// Static: 更新一次，使用多次
// Dynamic: 不断更新，使用多次
// Stream: 更新一次，最多使用几次
// Draw: application upload to GPU
// Read: GPU copy to application
// Copy: Draw and Read
glBufferData(GL_ARRAY_BUFFER, dataSize, vertices, GL_STATIC_DRAW);

// it is safe to delete after copying data to VBO
delete [] vertices;

// delete VBO when program terminated
glDeleteBuffers(1, &vboId);
```

2. 过去的使用方式：不同的 API  开启/关闭 不同的顶点属性
```c
glUseProgram(progId);

// bind VBOs for vertex array and index array
glBindBuffer(GL_ARRAY_BUFFER, vboId1);            // for vertex attributes
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, vboId2);    // for indices

glEnableClientState(GL_VERTEX_ARRAY);             // activate vertex position array
glEnableClientState(GL_NORMAL_ARRAY);             // activate vertex normal array
glEnableClientState(GL_TEXTURE_COORD_ARRAY);      // activate texture coord array

// do same as vertex array except pointer
glVertexPointer(3, GL_FLOAT, stride, offset1);    // last param is offset, not ptr
glNormalPointer(GL_FLOAT, stride, offset2);
glTexCoordPointer(2, GL_FLOAT, stride, offset3);

// draw 6 faces using offset of index array
glDrawElements(GL_TRIANGLES, 36, GL_UNSIGNED_BYTE, 0);

glDisableClientState(GL_VERTEX_ARRAY);            // deactivate vertex position array
glDisableClientState(GL_NORMAL_ARRAY);            // deactivate vertex normal array
glDisableClientState(GL_TEXTURE_COORD_ARRAY);     // deactivate vertex tex coord array

// bind with 0, so, switch back to normal pointer operation
glBindBuffer(GL_ARRAY_BUFFER, 0);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);
```

3. OpenGL 2.0 + 使用方式：同一个 API 开启/关闭 不同的顶点属性
```c
glUseProgram(progId);

// bind VBOs for vertex array and index array
glBindBuffer(GL_ARRAY_BUFFER, vboId1);            // for vertex coordinates
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, vboId2);    // for indices

glEnableVertexAttribArray(attribVertex);          // activate vertex position array
glEnableVertexAttribArray(attribNormal);          // activate vertex normal array
glEnableVertexAttribArray(attribTexCoord);        // activate texture coords array

// set vertex arrays with generic API
glVertexAttribPointer(attribVertex, 3, GL_FLOAT, false, stride, offset1);
glVertexAttribPointer(attribNormal, 3, GL_FLOAT, false, stride, offset2);
glVertexAttribPointer(attribTexCoord, 2, GL_FLOAT, false, stride, offset3);

// draw 6 faces using offset of index array
glDrawElements(GL_TRIANGLES, 36, GL_UNSIGNED_BYTE, 0);

glDisableVertexAttribArray(attribVertex);         // deactivate vertex position
glDisableVertexAttribArray(attribNormal);         // deactivate vertex normal
glDisableVertexAttribArray(attribTexCoord);       // deactivate texture coords

// bind with 0, so, switch back to normal pointer operation
glBindBuffer(GL_ARRAY_BUFFER, 0);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);
```

4. 更新 VAO
```c
// 方法 1：重新向 GPU 上传数据（缺点：应用程序和 GPU 要存储 2 份数据，每次更新都要占用带宽）
glBufferData(GL_ARRAY_BUFFER, dataSize, vertices, GL_STATIC_DRAW);

// 方法 2：通过映射 GPU 上缓存数据地址到应用程序的缓存地址
//        将应用程序对地址的操作用于对 GPU 缓存操作，达到在应用程序控制 GPU 缓存的效果

// bind then map the VBO
glBindBuffer(GL_ARRAY_BUFFER, vboId);

// Option: GL_READ_ONLY, GL_WRITE_ONLY, GL_READ_WRITE
// 如果 GPU 正在使用这个 buffer，将会返回 NULL
float* ptr = (float*)glMapBuffer(GL_ARRAY_BUFFER, GL_WRITE_ONLY);

// if the pointer is valid(mapped), update VBO
if(ptr)
{
    updateMyVBO(ptr, ...);          // custom function modify buffer data
    glUnmapBuffer(GL_ARRAY_BUFFER); // unmap it after use it's return GLboolean
}
```



# 三、Tessellation 曲面细分

Tessellation（曲面细分，可选阶段）：
当 Tessellation 阶段存在时，只能给管线提供 GL_PATCHES 类型的图元
当 Tessellation 阶段不存在时，不能提供 GL_PATCHES 类型的图元

- Tessellation Control Shader（TCS，细分控制着色器）
- Tessellation Primitive Generation（细分图元生成）
- Tessellation Evaluation Shader（TES，细分求值着色器）



# 引用

1. [Render To Texture](http://www.paulsprojects.net/opengl/rtotex/rtotex.html)
2. [OpenGL Vertex Buffer Object (VBO)](http://www.songho.ca/opengl/gl_vbo.html)
3. [How to choose between GL_STREAM_DRAW or GL_DYNAMIC_DRAW?](https://stackoverflow.com/questions/8281653/how-to-choose-between-gl-stream-draw-or-gl-dynamic-draw)