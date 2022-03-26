---
title: games202代码框架简述
date: 2021-08-27 17:09:53
tags: 学习
---


# 前言
本人对 js 和 WebGL 并不熟悉。所以若有谬误，欢迎斧正。

顺带一提本人对WebGL的内容的理解全部基于作业的PDF文档。

文章参考知乎高赞回答 https://zhuanlan.zhihu.com/p/396924653

# 代码结构

![1](images/games202_1.png)
主要是 assets lib src三个目录，接下来以此介绍

## assets

顾名思义，是放资产的地方。这个地方放你的模型资源。
![2](images/games202_2.png)
一个模型内容只有一个obj，存放模型顶点数据。一个mtl和一个png来存储uv贴图信息。

## lib

顾名思义，这是个图书馆(bushi)，你用的所有静态库。
![3](images/games202_3.png)

### dat.gui.js
一个轻量 GUI 框架，主要为了实现参数调整的GUI功能
链接 https://github.com/dataarts/dat.gui

### gl-matrix-min.js

矩阵及向量库
链接 https://glmatrix.net/

### imgui.umd.js 以及 imgui_impl.umd.js

这两个 GUI 的库在代码中并没有使用，删除这两个文件运行页面也不会报错。我猜测可能是开始想用这个库来做GUI，后来转向了 dat.GUI，然而文件并没有删除。
发现一个很有意思的事情是，闫老师在视频中展示的大学期间的作品的UI和这两个库的表现非常相像。(games101老师讲曲面细分那一期)
合理推测原框架是老师在大学期间搭的，然后助教同学整理作业的时候换成了自己更喜欢的库。但是原来的库并没有删除。

### MTLLoader.js

实现了 .mtl 文件的加载功能，被 OBJLoader.js 使用
.mtl 指的是 Material Library File，带有材质参数定义，类似于Obj格式。
实现了贴图自由


### OrbitControls.js

three.js 的相机控件，可以实现场景的缩放、旋转等操作，之所以不在 three.js 中，而是单独存在于一个 js 文件中，是因为这个存在于 three.js 的 example 目录下的实例代码，并不属于 three.js 的发布库。

### OBJLoader.js

同样是 three.js 中的示例代码，实现了 .obj 文件的加载功能。

### three.js

three.js 封装了 WebGL 的 API， 提供了一套更加易于使用的接口

## src

![4](images/games202_4.jpg)

### engine.js

逻辑主入口，设置了显示环境，以及 Camera 等设置。添加了 Light 和 场景中的 Objects，在逻辑的最后面创建了 UI，代码比较浅显易懂。
在逻辑开始的地方，使用 canvas.getContext('webgl') 创建了一个 WebGLRenderingContext，这也是为什么后面可以在这个 canvas 上使用 WebGL API 的原因。

### lights

#### DirectionalLight.js

定义一个平行光

``` js{.line-numbers}
constructor(
    lightIntensity,
    lightColor,
    lightPos,
    focalPoint,
    lightUp,
    hasShadowMap,
    gl
  )

```

前面几个参数都很好理解，但是第5行的的 focalPoint 焦点可能会让人摸不着头脑，为什么平行光还有焦点？
其实仔细想想会很容易理解，因为我们定义的光源还是一个点光源，在距焦点处在放一个凸透镜他就变成平行光了。
然后下面lightUp就定义了平行光的方向。
然后就是有无ShadowMap


#### PointLights.js

定义一个点光源
``` js{.line-numbers}
constructor(
    lightIntensity, 
    lightColor,
    hasShadowMap, 
    gl)

```
除了设置光源方向的和平行光没有什么不同

### loads.js

关于加载shader和模型的相关函数

#### loadOBJ.js
