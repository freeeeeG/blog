---
title: games101学习笔记_1
date: 2021-08-14 11:40:36
tags: 
- games101
- 学习
---
# 配置环境
## 环境
系统：windows
IDE：vs
并无使用闫老师的环境，理由是想折腾。
没有使用vscode，是因为每次写完代码都要用cmake重新编译一次，debug好麻烦。
### 配置vs
c++标准: 17
![1](/images/games101/games101_1_1.png)
前四个作业使用了opencv和Eigen3这两个第三方库，所以需要手动添加。
Eigen3还好，nuget包上直接下下来就好了只是要注意头文件写成(自行使用nuget)
```
#include <Eigen/Eigen>
```
而不是作业框架中的那样。
opencv就比较麻烦一点不能nuget一步到位。不太清楚为什么不可以。
网上的方法都是自己配置link。我这里也是
先include配置路径
![2](/images/games101/games101_1_2.png)
然后是library的路径
![3](/images/games101/games101_1_3.png)
最后再去配一下link里面的input就好了。不过这里要注意的点是两个lib文件一个有d一个没有d分别对应debug模式和release模式。
![4](/images/games101/games101_1_4.png)
以上，环境算是配好了。前四个作业比较麻烦，因为代码中的矩阵和显示用这两个库写起来挺方便的。之后老师出于些原因，自己手撸了一个，就不用我们配环境了。

# 作业一
作业零没什么内容跳了。
直接开整作业一。
先让我们康康这个作业框架里有什么
## 代码总览
![5](/images/games101/games101_1_5.png)
哦！我亲爱的老弟，这是一个天杀的所以程序都有的main，一个光栅化（rasterizer），还有一个三角形(Triangle)的类
### main函数
先让我们康康这个main里面有什么
![6](/images/games101/games101_1_6.png)
emmmmmm
上面先是闫老师在课提到的mvp模型对应的三个方法
然后定义了一个光栅化程序，眼睛的位置。
然后下面两个值比较重要，pos记录所有空间中的点，ind记录空间中那三个点形成一个三角形。
定义好之后，就扔到光栅化中去。这一步我们可以想象成，建模师把模型修好了现在要扔给计算器（就是那个r）去跑。
if那一行对应命令行操作的先不管。
我们现在打开while先康康
![7](/images/games101/games101_1_7.png)
第一行先不管，作业一暂时没有用到深度测试和颜色填充（所以并没有什么软用在这）
然后进行mvp
最后光栅化成像（r.draw）
成好像之后就用opencv输出。里面的参数不太用纠结。就是一些显示的画布大小（700*700），一些图像显示模式（CV_32FC3），一些内容（就是r里的玩意）
最后在显示到屏幕上就搞定了。
下面那个是控制我们模型角度的。
好了，理解我们大概在干什么之后开始康康我们具体是怎么实现mvp的吧。
#### view函数
``` C++{.line-numbers}
Eigen::Matrix4f get_view_matrix(Eigen::Vector3f eye_pos)
{
    Eigen::Matrix4f view = Eigen::Matrix4f::Identity();
    Eigen::Matrix4f translate;
    translate << 1, 0, 0, -eye_pos[0], 0, 1, 0, -eye_pos[1], 0, 0, 1,
        -eye_pos[2], 0, 0, 0, 1;

    view = translate * view;

    return view;
}

```
第3行定义一个单位矩阵（全是1，很嗨恐怖的那种）；
然后我们把所有东西都平移一下，移动到以摄像机为原点。
所以写了一个仿射矩阵，因为是让摄像机为原点。所以所有东西都移动-eye_pos。
这就搞定了
#### model函数
``` C++{.line-numbers}
Eigen::Matrix4f get_model_matrix(float rotation_angle)
{
    Eigen::Matrix4f model = Eigen::Matrix4f::Identity();

    // TODO: Implement this function
    // Create the model matrix for rotating the triangle around the Z axis.
    // Then return it.
    Eigen::Matrix4f rotate;
    float radian = rotation_angle/180.0*MY_PI;
    rotate << cos(radian), -1*sin(radian), 0, 0,
              sin(radian), cos(radian), 0, 0,
              0, 0, 1, 0,
              0, 0, 0, 1;//单纯实现了关于z轴的旋转矩阵
    model = rotate * model; 
    return model;
}
```
这并不复杂
这里就不推到了（bushi）
写一个物体绕z轴旋转angle的矩阵就好了。
#### projection
``` C++{.line-numbers}
Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio,float zNear, float zFar)
{
    // Students will implement this function
    // TODO: Implement this function
    // Create the projection matrix for the given parameters.
    // Then return it.
    Eigen::Matrix4f projection = Eigen::Matrix4f::Identity(); 
    Eigen::Matrix4f P2O = Eigen::Matrix4f::Identity();//将透视投影转换为正交投影的矩阵
    P2O << zNear, 0, 0, 0,
        0, zNear, 0, 0,
        0, 0, zNear + zFar, (-1)* zFar* zNear,
        0, 0, 1, 0;// 进行透视投影转化为正交投影的矩阵
    float halfEyeAngelRadian = eye_fov / 2.0 / 180.0 * MY_PI;
    float t = zNear * std::tan(halfEyeAngelRadian);//top y轴的最高点
    float r = t * aspect_ratio;//right x轴的最大值
    float l = (-1) * r;//left x轴最小值
    float b = (-1) * t;//bottom y轴的最大值
    Eigen::Matrix4f ortho1 = Eigen::Matrix4f::Identity();
    ortho1 << 2 / (r - l), 0, 0, 0,
        0, 2 / (t - b), 0, 0,
        0, 0, 2 / (zNear - zFar), 0,
        0, 0, 0, 1;//进行一定的缩放使之成为一个标准的长度为2的正方体
    Eigen::Matrix4f ortho2 = Eigen::Matrix4f::Identity();
    ortho2 << 1, 0, 0, (-1)* (r + l) / 2,
        0, 1, 0, (-1)* (t + b) / 2,
        0, 0, 1, (-1)* (zNear + zFar) / 2,
        0, 0, 0, 1;// 把一个长方体的中心移动到原点
    Eigen::Matrix4f Matrix_ortho = ortho1 * ortho2;
    projection = Matrix_ortho * P2O;
    return projection;
}
```
9到12行进行透视投影转化为正交投影的矩阵
![8](/images/games101/games101_1_8.png)
原点是我们的eyes_pos移动到原点之后的点，也就是我们眼睛看的方向。
n是我们视点到屏幕的距离。
任意一点到屏幕上的xy的变化我们用三角形很容易得出关系是
他的坐标变为(x′,y′) = (x,y)*n/z
这是通过透视显示在屏幕上的效果。这个显而易见。
但是我现在是要把透视矩阵变化为正交矩阵。
所以我们得像办法确定z变化之后在空间中的位置。
这时不够聪明的金针菇就要说了这不是很简单吗。z不变就好了。
确实这个面没有任何问题,但是你之后稍微转一点角度,因为z没变xy变了,会使得物体形变.
