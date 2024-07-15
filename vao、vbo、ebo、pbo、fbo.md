### 背景

以正方形为例，没有vao、vbo和ebo之前，我们如何绘制一个正方形呢？

```
    LFloat7 vertexTriangle[] = {
            {-0.5, 0.1, -0.1, 1.0, 0.0, 0.0, 1.0},
            {-0.5, 0.9, -0.1, 0.0, 1.0, 0.0, 1.0},
            {0.5,  0.1, -0.1, 0.0, 0.0, 1.0, 1.0},
            {0.5,  0.9, -0.1, 1.0, 0.0, 0.0, 1.0},
    };
    glEnableClientState(GL_VERTEX_ARRAY);
    glEnableClientState(GL_COLOR_ARRAY);

    glVertexPointer(3, GL_FLOAT, sizeof(LFloat7), vertexTriangle);
    glColorPointer(4, GL_FLOAT, sizeof(LFloat7), &vertexTriangle[0].r);

    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);

    glDisableClientState(GL_VERTEX_ARRAY);
    glDisableClientState(GL_COLOR_ARRAY);
```

以glVertexPointer(3, GL_FLOAT, sizeof(LFloat7), vertexTriangle);为例，该行代码做了两件事情

1. 向GPU提交顶点数据vertexTriangle
2. 向GPU提交顶点数据的解析方式， 数据大小3， 数据类型FLOAT， 步长为sizeof(LFloat7)

然后，通过glDrawArrays绘制顶点数据.  

这样做存在一个问题：每次绘制都需要从CPU向GPU提交数据，效率不高，也没有必要。  
因此，创造了VBO，用于解决这个问题。  

### VBO

Vertex Buffer Object
初始化时，向使用VBO向CPU提交一次数据，之后数据就被缓存在GPU中，无需CPU再向GPU提交数据。绘制时，直接使用GPU中的缓存数据。
因此，整体上，VBO数据分三个阶段

1. 初始化阶段，创建VBO，并向GPU提交顶点数据
2. 绘制阶段，使用VBO的数据
3. 销毁阶段，释放VBO的数据

上述代码使用VBO的话，改造如下

#### 初始化

```
    //顶点数据
    LFloat7 vertexTriangle[] = {
            {-0.5, 0.1, -0.1, 1.0, 0.0, 0.0, 1.0},
            {-0.5, 0.9, -0.1, 0.0, 1.0, 0.0, 1.0},
            {0.5,  0.1, -0.1, 0.0, 0.0, 1.0, 1.0},
            {0.5,  0.9, -0.1, 1.0, 0.0, 0.0, 1.0},
    };
    //创建VBO
    Gluint vbo;
    glGenBuffers(1, &vbo);
    //绑定VBO数据（向GPU提交该VBO所绑定的数据）
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertexTriangle), vertexTriangle, GL_STATIC_DRAW);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
```

#### 绘制

绘制改造如下

```
     //激活VBO数据
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glEnableClientState(GL_VERTEX_ARRAY);
    glEnableClientState(GL_COLOR_ARRAY);

    //声明VBO数据的解析信息
    float* vertexAddress = (float*)0;
    float* colorAddress = (float*)12;
    glVertexPointer(3, GL_FLOAT, sizeof(LFloat7), vertexAddress);
    glColorPointer(4, GL_FLOAT, sizeof(LFloat7), colorAddress);
    //绘制
    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);

    glDisableClientState(GL_VERTEX_ARRAY);
    glDisableClientState(GL_COLOR_ARRAY);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
```

此处，vertexAddress和colorAddress稍微解释下，
vertexAddress的0是因为，顶点数据位于每帧数据的0 * sizeof(FLOAT) byte位,  也即是0，
同理， 颜色数据位于每帧数据的3 * sizeof(FLOAT) byte位,  也即是12 

##### 释放VBO

```
glDeleteBuffers(1, &vbo);
```

### VBO/EBO/PBO

三者基本一致，都是把数据存到GPU，减少CPU向GPU提交数据的次数，从而提高渲染效率。  
三者都是存数据，只是所存储的数据不同。

- VBO：存放顶点数据，比如顶点、颜色、法线，GL_ARRAY_BUFFER
- EBO: 存放顶点索引index数据, GL_ELEMENT_ARRAY_BUFFER
- PBO：存放像素数据

#### 向GPU提交数据

```
GLuint vboId;
//生成vbo id
glGenBuffers(1, &vboId);
//绑定vobId， 也即是， 后续操都针对该vboId
glBindBuffer(GL_ARRAY_BUFFER, vboId);
//向GPU提交该vbo的数据
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
//数据提交完，解绑vboId
glBindBuffer(GL_ARRAY_BUFFER, 0);
```

EBO、PBO类似，只是类型不同，GL_ARRAY_BUFFER、GL_STATIC_DRAW。

#### 绘制时使用数据

```
glBindBuffer(GL_ARRAY_BUFFER, vboId);
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (const void *)0);


glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_SHORT, (const void *)0);
```

#### 绘制结束，释放

```
glDeleteBuffers(1, &vboId);
```

### VAO

从上述的操作看，虽然减少了向GPU提交数据的操作，但是每次绘制仍然需要调用glVertexAttribPointer方法，声明VBO数据的使用信息，略显繁琐。因此，有了VAO。

```
//创建VAO
GLuint vaoId;
glGenVertexArrays(1, &vaoId);
glBindVertexArray(vaoId);

//关联vbo
GLuint vboId;
//生成vbo id
glGenBuffers(1, &vboId);
//绑定vobId， 也即是， 后续操都针对该vboId
glBindBuffer(GL_ARRAY_BUFFER, vboId);
//向GPU提交该vbo的数据
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
//数据提交完，解绑vboId
glBindBuffer(GL_ARRAY_BUFFER, 0);

//解绑vao
glBindVertexArray(0);
```

绘制

```
//激活vao
glBindVertexArray(vaoId);
//绘制三角形
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_SHORT, (const void *)0);
//解绑vao
glBindVertexArray(0);
```

### 参考文章

- [熟悉 OpenGL VAO、VBO、FBO、PBO 等对象，看这一篇就够了](https://cloud.tencent.com/developer/article/1893989)
- [Android OpenGL 渲染图像读取哪家强](https://cloud.tencent.com/developer/article/1739511)
- [PBO是OpenGL最高效的像素拷贝方式吗？](https://cloud.tencent.com/developer/article/2003936)
- [OpenGL学习脚印: 绘制一个三角形](https://www.cnblogs.com/zgyijg/p/14817909.html)
