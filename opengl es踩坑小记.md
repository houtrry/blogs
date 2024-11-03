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

#### 2. glsl编译error，报reason is FLEX: Unknown char 
具体如下
>reason is FLEX: Unknown char    
>          ERROR: 0:22: '' : Syntax error:  syntax error   
>          ERROR: 1 compilation errors.  No code generated.
  
glsl文件内容如下：
```
#version 300 es

precision mediump float;
in vec2 v_texCoord;
layout(location = 0) out vec4 outColor;
uniform sampler2D s_TextureMap;

void main() {
//    outColor = texture(s_TextureMap, v_texCoord);
//    flat gray;
//    gray  = float(texture(s_TextureMap, v_texCoord).r);
    float gray = texture(s_TextureMap, v_texCoord).r;
    if(132.0 / 255.0 == gray) {
        outColor = vec4(1.0, 1.0, 1.0, 1.0);
    } else if (1.0 == gray) {
        outColor = vec4(0.49725, 0.847059, 0.917647, 1.0);
    } else {
        outColor = vec4(0.0, 0.447, 1.0, 1.0);
    }

//    outColor = vec4(gray, gray, gray, 1.0);
}
```
删除注释后，编译正常，内容如下。
```
#version 300 es

precision mediump float;
in vec2 v_texCoord;
layout(location = 0) out vec4 outColor;
uniform sampler2D s_TextureMap;

void main() {
    float gray = texture(s_TextureMap, v_texCoord).r;
    if(132.0 / 255.0 == gray) {
        outColor = vec4(1.0, 1.0, 1.0, 1.0);
    } else if (1.0 == gray) {
        outColor = vec4(0.49725, 0.847059, 0.917647, 1.0);
    } else {
        outColor = vec4(0.0, 0.447, 1.0, 1.0);
    }

//    outColor = vec4(gray, gray, gray, 1.0);
}
```
出现这个问题，应该检查glsl文件内容中是否有非法字符

#### 3. 预览单通道图像内容发生了偏移
如下图：
![Screenshot_20241103_163841](https://github.com/user-attachments/assets/5802081d-bf63-4cd2-83cc-afb3890cf85e)

添加如下设置后，代码预览正常
```
glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
```
GL_UNPACK_ALIGNMENT默认值是4，[GL_UNPACK_ALIGNMENT说明](https://blog.csdn.net/keneyr/article/details/102727363)
修改后预览如下：
![Screenshot_20241103_163931](https://github.com/user-attachments/assets/4ceaa38e-6cb2-4be8-920b-e9367bcff084)
