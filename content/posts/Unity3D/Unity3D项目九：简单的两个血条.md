---
title: "Unity3D项目九：简单的两个血条"
date: 2020-04-28T00:14:10+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# 简单血条 IMGUI和UGUI

@[toc]
## 作业要求

- 分别使用 IMGUI 和 UGUI 实现

- 使用 UGUI，血条是游戏对象的一个子元素，任何时候需要面对主摄像机

- 分析两种实现的优缺点

- 给出预制的使用方法

  

## 效果展示

项目地址 -> [传送门](https://github.com/Tifinity/Unity3DStudy-master/tree/master/%E9%A1%B9%E7%9B%AE%E4%B9%9D%EF%BC%9A%E7%AE%80%E5%8D%95%E4%B8%A4%E4%B8%AA%E8%A1%80%E6%9D%A1/Assets )

视频连接 -> [传送门]( https://www.bilibili.com/video/av76262413/)



## 具体实现

### UGUI实现

#### 制作血条

先随便创建一个Cube，然后在Cube下创建子物体Canvas：右键->UI->Canvas；

接着在Canvas下创建Slider：右键->UI->Slider；

场景中一个白色的大框就是画布Canvas，画布中有一个滑动条Slider，项目结构树中可以看到如下结构。展开Slider可以看到组成滑动条的三个组件：Background-背景颜色，Fill Area-填充颜色，Handle SliderArea-滑动柄。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191119153708572.jpg)

右边检查器中可以修改滑动条的各个属性，拖动value可以看到滑动条随之移动：

![在这里插入图片描述](https://img-blog.csdnimg.cn/201911191537202.jpg)

现在删掉滑动柄Handle SliderArea，然后将Fill中的图像->颜色改为你喜欢的颜色，将Fill Area和Fill中Rect Transform中的左右都设为0，不然血条会超出填充框。

一个简单的血条现在就差不多做好了。

很重要的一点：千万不要乱改Rect Transform的其他固定值，调大小用缩放来调。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191119153725600.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)



#### 摆放血条

现在血条静止的在场景中，我们需要让它跟随摄像机移动。

Canvas->Canvas画布->Render Mode渲染模式，这里有三种渲染模式：

- Screen Space - Overlay:默认的渲染模式，将UI元素放置在场景顶部渲染的屏幕，画布会自动更改大小匹配屏幕。此模式Canvas位置大小不可改变，但可以通过移动父物体来间接改变，Canvas的起始位置就是父物体的位置，大小和设置的显示屏幕大小有关，更改不同的显示大小，Canvas的形状大小也将会发生改变。此模式不需要UI相机，Canvas上的内容将显示在所有游戏物体之前。

- Screen Space - Camera：此模式Canvas位置大小不可改变，需要指定UI相机定，可以将Main Camera挂载到上面观察效果，画布上的内容将一直显示在相机视野里，且
显示在所有游戏物体之前。

- World Space： 画布行为与场景中的其他任何对象一样，UI元素将放置在其他对象的前面或后面渲染。**画布大小和位置任意设置**，这对于意在成为世界一部分的用户界面非常有用。 

选择World Space，再将Main Camera拖到事件摄像机的位置，这时画布的Rect Transform可以修改了，在修改画布和血条的位置和比例到合适的位置。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191119153813227.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)



#### 控制血条

按照课程网站教程，将血条挂在官方人物上，顺便一提，新版本的Unity3D要在资源商店里手动导入Standard Asserts，再里面找到Characters 的Prefabs就可以看到教程中的人物了。

在这里我还是使用了之前作业做的人物。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191119153820295.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

简单的写了控制血条的代码：

~~~C#
using UnityEngine;
using UnityEngine.UI;

public class UGUI_H: MonoBehaviour {
    public Slider HPStrip;
    public Image fill;
    public float HP = 10f;
    private float tmpHP;
    void Awake() {
        HPStrip.value = HPStrip.maxValue = HP;
        tmpHP = HP;
        fill.color = Color.green;
    }
    void Update() {
        HP = Mathf.Lerp(HP, tmpHP, 0.05f);
        HPStrip.value = HP;
        if (HP<=5) {
            fill.color = Color.red;
        }
    }
    public void SetHP(int flag, float damage) {
        if(flag == 1 && HP <= 10) {
            tmpHP += damage;
        } else if(flag == 0 && HP > 0) {
            tmpHP -= damage;
        }
        
    }
    public float GetHP() {
        return HP;
    }
}
~~~



不过老师提出了以下问题：

>如果你在生产中使用上述血条，足以证明你是入门级别程序员。为了显示血条，每个图片从 pixel 单位映射到 world space，再投影到摄像机空间，在变成屏幕空间（pixel 单位）。这需要多少算力？ 
>
>因此，能用 Srceen Space 解决的问题绝不用 World Space， 能用 Overlay 解决问题绝不用 Camera！
>
>而血条恰恰是 Srceen Space - Overlay 能决解的哦！提示，阅读脚本手册 RectTransform

暂时还没想到如何解决，基本思路还是先work起来再改进，这个问题留待以后解决。



### IMGUI

用代码实现即可。

~~~C#
using UnityEngine;

public class IMGUI_H : MonoBehaviour {
    public float HP = 1.0f;
    private float tmpHP;

    private Rect HealthBar;

    void Start() {
        HealthBar = new Rect(50, 50, 200, 20);
        tmpHP = HP;
    }

    void OnGUI() {
        HP = Mathf.Lerp(HP, tmpHP, 0.05f);
        GUI.HorizontalScrollbar(HealthBar, 0.0f, HP, 0.0f, 1.0f);
    }

    public void SetHP(int flag, float damage) {
        if (flag == 1) {
            tmpHP = tmpHP + 0.1f > 1.0f ? 1.0f : tmpHP + 0.1f;
        }
        else if (flag == 0) {
            tmpHP = tmpHP - 0.1f < 0.0f ? 0.0f : tmpHP - 0.1f;
        }
    }

    public float GetHP() {
        return HP;
    }
}
~~~



### 两种方法比较

IMGUI：

优点：

- 符合传统编程习惯


缺点：

- 效率低下，难以调试。

> 按 Unity 官方说法，IMGUI主要用于以下场景：
>
> - 在游戏中创建调试显示工具
> - 为脚本组件创建自定义的 Inspector 面板。
> - 创建新的编辑器窗口和工具来扩展 Unity 环境。

IMGUI不是我们想象中的与用户交互的UI，UGUI更像我们玩游戏的时候见到的UI。

UGUI：

优点：

- UGUI出现了锚点的概念，更方便屏幕自适应。 

- 所见即所得，设计师也能参与程序开发
- UI 元素与游戏场景融为一体的交互
- 面向对象

缺点：

- 暂时没有使用上的缺点，因为还没有深入了解



### 预制体使用方法

两个预制体为IMGUI-H-Bar，UGUI-H-Bar，将其放在你的人物子对象中，在人物控制脚本中添加对应组件即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191119153833735.jpg)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191119153840182.jpg)



## 参考资料

[前辈博客](https://www.cnblogs.com/vmoor2016/p/6044941.html )

[课程网站]( https://pmlpml.github.io/unity3d-learning/09-ui.html )