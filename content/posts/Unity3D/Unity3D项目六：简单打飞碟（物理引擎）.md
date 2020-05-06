---
title: "Unity3D项目六：简单打飞碟（物理引擎）"
date: 2020-04-28T00:21:16+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# 简单打飞碟（物理引擎）

@[toc]
## 游戏规则与要求

- 游戏内容要求：
  1. 按 *adapter模式* 设计图修改飞碟游戏
  2. 使它同时支持物理运动与运动学（变换）运动

## 附加项目
[附加项目简单打靶](https://blog.csdn.net/Tifinity/article/details/102617434)

## 项目地址与演示视频

项目地址 -> [传送门?](https://github.com/Tifinity/Unity3DStudy-master/tree/master/%E9%A1%B9%E7%9B%AE%E5%85%AD%EF%BC%9A%E7%AE%80%E5%8D%95%E6%89%93%E9%A3%9E%E7%A2%9F%EF%BC%88%E7%89%A9%E7%90%86%E5%BC%95%E6%93%8E%EF%BC%89)

视频连接 -> [传送门?](https://www.bilibili.com/video/av70636842/)

演示视频与运动学版本无较大差别。

## 具体实现

在上次作业的基础上增加使用物理引擎的动作管理类，并在场景控制器中对两个动作管理类进行选择即可。

- 首先，若要使用物理引擎，我们需要使用刚体（Rigidbody）组件，刚体组件不能通过Update()函数来刷新，Update()的调用速率默认是60次/秒，受到机器性能和被渲染物体的影响，但是物理引擎渲染是一个固定的时间，是可以设置的。

   在Edit->ProjectSetting->Time:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191017125012395.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

  所以对于刚体的更新我们需要用FixedUpdate()函数来实现。

- 在SSActionManager.cs中，加上FixedUpdate，使动作管理类可以适配物理引擎：

  ```c#
  //对于字典中每一个pair，看是执行还是删除
  foreach (KeyValuePair<int, SSAction> kv in actions) {
      SSAction ac = kv.Value;
      if (ac.destroy) {
          waitingDelete.Add(ac.GetInstanceID());
      }
      else if (ac.enable) {
          ac.Update();
          ac.FixedUpdate(); //-----适配物理引擎
      }
  }
  ```

  

- 在SSAction.cs中同样加上FixedUpdate()。

  ```C#
  public virtual void Update() {
      throw new System.NotImplementedException();
  }
  //-----适配物理引擎
  public virtual void FixedUpdate() {
      throw new System.NotImplementedException();
  }
  ```

  

- PhysisDiskFlyAction.cs

  与DiskFlyAction大同小异，将所有操作都放进FixedUpdate()，当飞碟落到屏幕外时将飞碟的速度清空。Start()中对飞碟施加一个冲击力（Impulse）。

  ```c#
  public class PhysisDiskFlyAction : SSAction {
      private Vector3 start_vector;                              
      public float power;
      private PhysisDiskFlyAction() { }
      public static PhysisDiskFlyAction GetSSAction(int lor, float power) {
          PhysisDiskFlyAction action = CreateInstance<PhysisDiskFlyAction>();
          if (lor == -1) action.start_vector = Vector3.left * power;
          else action.start_vector = Vector3.right * power;
          action.power = power;
          return action;
      }
  
      public override void Update() { }
  
      public override void FixedUpdate() {
          if (transform.position.y <= -10f) {
              gameobject.GetComponent<Rigidbody>().velocity = new Vector3(0, 0, 0);
              this.destroy = true;
              this.callback.SSActionEvent(this);
          }
      }
  
      public override void Start() {
          gameobject.GetComponent<Rigidbody>().AddForce(start_vector*3, ForceMode.Impulse);
      }
  }
  ```

  

- FlyActionManager.cs

  在动作管理器中加上使用物理引擎的动作并重载DiskFly函数，当不使用物理引擎时启用isKinematic运动学变换，当使用物理引擎时关闭运动学变换：

  ```c#
  //飞碟飞行
  public void DiskFly(GameObject disk, float angle, float power) {
      disk.GetComponent<Rigidbody>().isKinematic = true;
      int lor = 1;
      if (disk.transform.position.x > 0) lor = -1;
      fly = DiskFlyAction.GetSSAction(lor, angle, power);
      this.RunAction(disk, fly, this);
  }
  
  public void DiskFly(GameObject disk, float power) {
      disk.GetComponent<Rigidbody>().isKinematic = false;
      int lor = 1;
      if (disk.transform.position.x > 0) lor = -1;
      fly_ph = PhysisDiskFlyAction.GetSSAction(lor, power);
      this.RunAction(disk, fly_ph, this);
  }
  ```

  

- 最后给飞碟的预制体加上刚体组件即可，其他代码都不需要改动。

- 在FirstController中可切换：

  ```c#
  action_manager.DiskFly(disk, angle, power);//运动学变换
  //action_manager.DiskFly(disk, power);//物理引擎
  ```



## 总结

本次作业主要是了解了物理引擎在Unity3D中的使用，包括FixedUpdate函数，刚体组件，对游戏对象速度和力的操作，使用物理引擎就不需要模拟运动学的位置变换，使用复杂的公式计算。总的来说过程比较顺利，没有太大的阻碍。