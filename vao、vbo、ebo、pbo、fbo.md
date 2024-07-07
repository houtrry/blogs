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

在绘制过程中，我们可能还会遇到使用glDrawElements方法进行绘制的情况，这个时候，EBO就派上了用场
### EBO

但是VBO只解决了缓存顶点数据的问题，在使用VBO数据时，仍然需要向GPU提交顶点数据的解析方式，  
为了简化使用，我们创造了VAO
### VAO
  
   
   


- VBO可以单独使用，不依赖VAO、EBO  
- VAO依赖VBO使用，也即是，使用VAO时，必须使用VBO  
- EBO依赖VBO使用，也即是，使用EBO时，必须使用VBO  

VBO是VAO、EBO的基础

### 参考文章
- [熟悉 OpenGL VAO、VBO、FBO、PBO 等对象，看这一篇就够了](https://cloud.tencent.com/developer/article/1893989)
- [Android OpenGL 渲染图像读取哪家强](https://cloud.tencent.com/developer/article/1739511)
- [PBO是OpenGL最高效的像素拷贝方式吗？](https://cloud.tencent.com/developer/article/2003936)


