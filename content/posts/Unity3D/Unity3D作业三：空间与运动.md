---
title: "Unity3D作业三：空间与运动"
date: 2020-04-28T01:23:11+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# Unity3D作业二：空间与运动

**作业内容**

1、简答并用程序验证

- 游戏对象运动的本质是什么？

  运动的本质是游戏对象位置，欧拉角，形状的改变。

- 请用三种方法以上方法，实现物体的抛物线运动。（如，修改Transform属性，使用向量Vector3的方法…）

  1. 使用Vector3修改position

     ```c#
     using System.Collections;
     using System.Collections.Generic;
     using UnityEngine;
     
     public class move : MonoBehaviour {
         private float speed = 0.25f;//初速度
         private float acceleration = 0.20f;//加速度
         // Start is called before the first frame update
         void Start() {
         }
         // Update is called once per frame
         void Update() {
             this.transform.position += Vector3.right * speed * Time.time;
             this.transform.position += Vector3.down * acceleration * Time.time * Time.time;
         }
     }
     ```

  2. 使用Translate

     ```c#
     void Update () {
     	transform.Translate (transform.right * speed * Time.fixedTime, Space.World);
     	transform.Translate (transform.down * acceleration * Time.fixedTime * Time.fixedTime, Space.World);
     }
     ```

  3. 计算出移动过程中的Transform.position，直接对其赋值：

     ```c#
     void Update () {
         float s = speed * Time.deltaTime;
         transform.position = Vector3.MoveTowards(transform.position, target, s);
     }
     ```

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921191402198.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

- 写一个程序，实现一个完整的太阳系， 其他星球围绕太阳的转速必须不一样，且不在一个法平面上。

    项目博客：[?](https://blog.csdn.net/Tifinity/article/details/100884365)

  

2. 编程实践

   项目博客：[?](https://blog.csdn.net/Tifinity/article/details/101114425)