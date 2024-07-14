#### 1. jni里使用AssertsManager读取asserts文件夹里的glsl文件出现数据重复
现象：
glsl文件内容正常，但是使用AssertsManager读出的数据里，字符串重了
比如，顶点着色器内容如下：
```
#version 300 es

layout(location = 0) in vec4 a_Position;
layout(location = 1) in vec4 a_Color;
out vec4 v_Color;
void main() {
    gl_Position = a_Position;
    v_Color = a_Color;
}
```
但是打印显示，读到的数据如下：
>#version 300 es  
>  
>layout(location = 0) in vec4 a_Position;  
>layout(location = 1) in vec4 a_Color;  
>out vec4 v_Color;  
>void main() {  
>    gl_Position = a_Position;  
>    v_Color = a_Color;  
>}  
>#version 300 es  
>  
>layout(location = 0) in vec4 a_Position;  
>layout(location = 1) in vec4 a_Color;  
>out vec4 v_Color;  
>void main() {  
>    gl_Position = a_Position;  
>    v_Color = a_Color;  
>}


也即是内容出现了两遍。  
最后发现，原因竟然是glsl代码的最后有换行符、空格，删除最后一个大括号}后的所有内容后，读取正常。
