---
title: "Unity3D作业四：材料与渲染练习"
date: 2020-04-28T01:23:49+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# 材料与渲染练习

先在这里推荐一个纹理材质的网站 -> [?](https://www.textures.com/download/pbr0233/133289)

现在开始我的学习过程ヽ(✿ﾟ▽ﾟ)ノ

首先在网站上随便找了一块免费的石头

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927150918643.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

往下可以看到一共有五张贴图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927150928848.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

选择分辨率为Medium（1024*1024），全部下载并放进Unity3D的素材中。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927150938937.jpg)

我们一张一张的来，创建一个新的材质球和一个球体对象。首先是Albedo贴图，将其拖到Albedo属性左边的贴图框中，球体变成了这个样子。因为这块石头的纹理比较简单，看起来就像一个黑色的球，仔细看能够看到它的纹路细节。Albedo贴图其实就是一张RGB的图片，你也可以点击Albedo右边的颜色编辑器来创建一个纯色的贴图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927150952684.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
这样的材质未必太过简陋，那么那些逼真复杂的纹理是怎么创建的呢，我们先不管金属度和平滑度，往下看︿(￣︶￣)︿

把Normal贴图加上，就是那张看起来是蓝紫色的。球体变成了这样，有了凹凸的纹理，更加逼真了。不过这个高低差不是真的，你会发现边缘还是一个完整的正圆形，说明其实没有真的凹下去或者凸出来，只是视觉上的效果。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151002134.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

下面放上官网的法线贴图效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151015804.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

然后我们加上高度贴图，这里有一个问题，高度贴图是灰度图，如果下载的格式是TIF，那么导入进Unity3D会变成天蓝色，需要用PS或者别的软件转一下格式为JPG才正常。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151032836.jpg)

这个素材加上高度贴图的效果感觉不是很明显，就不放图了，可以看官网的对比。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151040631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

关于法线贴图和高度贴图的异同点详见：[?](http://www.manew.com/thread-90210-1-1.html)

现在该Ambient Occlusion了，一般称为ao贴图，翻译为环境遮挡，加上之后可以看到更明显的阴影，下图左为无ao，右为有ao，可以看到沟壑的颜色明显变深了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151050839.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

还以最后一张，roughness贴图，在unity3D中想要使用的话需要做一些处理。因为我们的贴图是“粗糙度”，而unity中只有“平滑度”，并且为了节约资源，平滑度其实就是金属度的Alpha通道的值，需要用图像处理软件进行处理，所以此次我们暂时不使用roughness贴图了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151100703.jpg)

那么现在我们回到金属度和平滑度。我们都知道金属度控制材质表面金属感的程度，平滑度控制材质表面从橡胶到玻璃的光滑程度。

为了金属度的效果更加明显，我们将材质球换成纯白色。将材质球的金属度盒平滑度都拉到最高，你可以看到球体上“反射”出了天空盒，但是这真的是像现实一样的反射吗？

进入菜单栏->窗口->渲染->照明设置，在环境照明中将源改为红色，环境反射改为自定义，并在贴图中选择原来的天空盒。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151149543.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

发生了什么？即使天上没有天空盒，球体上也出现了反射的天空！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927151125693.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

其实，金属度的本质就是环境照明和环境反射的比例，而平滑度控制金属度的Alpha通道即透明度。

再随便调一调金属度和平滑度看看效果：

![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-mnDSzpMd-1569568101639)(T:\TH\大三上\3D游戏设计\4游戏对象与图形基础\材料与渲染练习\images\调整.jpg)\]](https://img-blog.csdnimg.cn/20190927151213696.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

|      | 金属度 | 平滑度 |
| ---- | ------ | ------ |
| 左上 | 0      | 0      |
| 左下 | 1      | 0      |
| 右上 | 0      | 1      |
| 右下 | 1      | 1      |

那么本次材料与渲染练习就到此为止了，继续努力吧**·~·**