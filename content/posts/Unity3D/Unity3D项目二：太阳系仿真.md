---
title: "Unity3D项目二：太阳系仿真"
date: 2020-04-28T01:21:11+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# Unity3D项目二：太阳系仿真

先上演示视频：[?](https://www.bilibili.com/video/av67946317/)

### 视角移动秘技

1. 飞行模式：按住鼠标右键 + WASD（或方向键）移动，QE可以缩放
2. 聚焦旋转：选中物体按F或者双击聚焦，然后按住ALT + 鼠标左键移动

### 物体旋转

围绕自身的旋转，参数为旋转轴 * 速度，注意只有一个参数

`this.transform.Rotate (Vector3.up * rotaSpeed);`

围绕某个点的旋转，参数为围绕的点，旋转轴，速度

`this.transform.RotateAround (Vector3.zero, Vector3.up, rotaSpeed);`

### 完成过程

首先创建九个球体排成一列，并挂上相应材质

我使用的材质链接：[?](https://tieba.baidu.com/p/4876471245?red_tag=2938874589)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916125906864.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

将游戏对象结构创建成这样，Salor是个空对象

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916125918115.jpg)

脚本是solar的组件，我使用的方法是将所有行星作为公有成员添加到脚本中，然后控制他们的旋转即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916125926600.jpg)

公转时为了实现在不同的法平面上，使用Vector3作为旋转轴，可以自由调整向量的方向，不过如果Y方向为负数则速度也需要为负数，否则会公转方向不同。

```c#
 /*公转*/
 mercury.RotateAround(this.transform.position, new Vector3(4, 10, 0), 47 * Time.deltaTime);
 venus.RotateAround(this.transform.position, new Vector3(2,10, 0), 35 * Time.deltaTime);
 earth.RotateAround(this.transform.position, new Vector3(1, -10, 0), -30 * Time.deltaTime);
 mars.RotateAround(this.transform.position, new Vector3(2, 10, 0), 24 * Time.deltaTime);
 jupiter.RotateAround(this.transform.position, new Vector3(2, -10, 0), -13 * Time.deltaTime);
 saturn.RotateAround(this.transform.position, new Vector3(1, -10, 0), -9 * Time.deltaTime);
 uranus.RotateAround(this.transform.position, new Vector3(2, 10, 0), 6 * Time.deltaTime);
 neptune.RotateAround(this.transform.position, new Vector3(1, -10, 0), -5 * Time.deltaTime);
 
```

自转的比较简单

```c#
/*自转*/
 earth.Rotate(Vector3.up * Time.deltaTime * 250);
 mercury.Rotate(Vector3.up * Time.deltaTime * 300);
 venus.Rotate(Vector3.up * Time.deltaTime * 280);
 mars.Rotate(Vector3.up * Time.deltaTime * 220);
 jupiter.Rotate(Vector3.up * Time.deltaTime * 180);
 saturn.Rotate(Vector3.up * Time.deltaTime * 160);
 uranus.Rotate(Vector3.up * Time.deltaTime * 150);
 neptune.Rotate(Vector3.up * Time.deltaTime * 140);
```

最后为了好看点把主摄像机的背景调成黑色

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916125936292.jpg)

并且给每个星球都加上拖尾渲染

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916125941526.jpg)