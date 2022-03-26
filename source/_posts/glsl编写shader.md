---
title: glsl编写shader
date: 2021-08-23 18:47:55
tags: 学习
---

# 何为shader
CPU端给GPU发送的一组指令，使其能在屏幕上着色（渲染图片）的程序。
这个过程发生在pipeline中的Vertex Processing（顶点渲染）和Fragment Processing(图元或者像素渲染)。

## 何为shader language
编写具体的Vertex Processing和Fragment Processing的逻辑。
glsl有着类C风格的代码（所以可读性不高）。
要注意很多变量是全局的所以阅读项目非常莫名其妙如果你不懂渲染的大概流程。

## shader setup
### 1.创造shader（Vertex 和 Fragment）
#### 总述
两个shader渲染的时候一般的顺序是先Vertex后Fragment。
这个应该很好理解，毕竟顶点都无上哪里去渲染像素。
然后两个shader在扔给openGL的时候里面的代码是遍历所有的顶点和像素的。所以我们不用再去写for循环，他帮我们做好了。

#### Vertex Shader
``` glsl
attribute vec3 aVertexPosition;(位置)
attribute vec3 aNormalPosition;(法线)
attribute vec2 aTextureCoord;(纹理坐标)
(attribute是只有Vertex中才有的变量)

uniform mat4 uModelViewMatrix;
uniform mat4 uProjectionMatrix;
（全局变量要考的）

avrying highp vec2 vTextureCoord;
avrying highp vec3 vFragPos;
avrying highp vec3 vNormal;
（highp是精度）
（avrying是Vertex 和Fragment共享的变量，你比如插值什么的）

void main() {
    vFragPos = aVertexPosition;
    vNormal = aNormalPosition;
    gl_Position = uProjectionMatrix*uModelViewMatrix*vec4(aVertexPosition,1.0);
    （不知道为什么后面要乘那个vec4的变量的同学，去看MVP补课）
    vTextureCoord = aTextureCoord;


}
```


#### Fragment Shader
``` glsl

uniform sampler2D uSampler;
uniform vec3 uKd;
（漫反射系数，我tm也不知道为什么叫kd，明明原名叫Lambert模型）
uniform vec3 uKs;
（镜面反射系数，我也tm不知道为什么叫ks，明明原名叫Phong模型）
uniform vec3 uLightPos;
（光源位置，动态光照不稀罕这玩意）
uniform vec3 uCameraPos;
（相机位置）
uniform float uLightIntensity;
（光照强度）
umiform int uTextureSample;
（纹理贴图）

avrying highp vec2 vTextureCoord;
avrying highp vec3 vFragPos;
avrying highp vec3 vNormal;
（同上）

void main() {
    vec3 color;
    if(uTextureSample == 1) {
        color = pow(texture2D(uSampler,vTextureCoord).rgb,vec3(2.2));
        （后面那个vec3是dleta值，搞那个色差用的，一般的都是<=3）
    }
    else{
        color = uKd;
    }
    （这一步有的就贴上，没有的就反射一下）
    vec3 ambient = 0.05*color;
    vec3 lightDir = normalize(uLightPos - vFragPos);
    vec3 normal = normalize(vNormal);
    float diff = max(dot(lightDir,normal),0.0);
    float light_atten_coff = uLightIntensity/length(uLightPos - vFragPos);
    vec3 diffuse =diff*light_atten_coff*color;

    vec3 viewDir = normalize(uCameraPos - vFragPos);
    float = spec = 0.0;
    vec3 reflectDir = reflect(-lightDir,normal);
    spec = pow(max(dot(viewDir,reflectDir),0.0).35.0)'
    vec3 specular = uKs*light_atten_coff*spec;

    gl_FrafColor = vec4(pow((ambient+diffuse +specular),vec3(1.0/2.2)),1.0);
}
```


### 2.编译shader
GLuint shader = glCreateShader(type);
(建立一个空的shader)
string str = textFileRead (filename);
(读取你写好的shader)
glShaderSource(shader, 1, &cstr2, NULL);
glCompileShader(shader);
（这两步就是扔给openGL编译）
### 3.添加到program
GLuint program = glCreateProgram();
glAttachShader(program,vertexShader);
glAttachShader(program,fragmentShader);
简单讲讲为什么要有这个program。
因为很多时候vertex 和fragment是相互作用的，
你比如一个三角形的三个顶点分别记录了一种颜色，其中覆盖的每一片像素通过他们的插值显示颜色。
这就要他们两个shader之间的交互了。
### 4.连接program
glLinkProgram(program);
就链接。。。
链接完了之后要做一些判断
如果成功就返回，不成功报个错什么的。
### 5.使用program
就鞥用就好了。
 

