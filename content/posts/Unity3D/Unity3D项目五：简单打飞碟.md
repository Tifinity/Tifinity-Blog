---
title: "Unity3D项目五：简单打飞碟"
date: 2020-04-28T00:22:01+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# 打飞碟

@[toc]
## 游戏规则与要求

- 游戏内容要求：
  1. 游戏有 n 个 round，每个 round 都包括10 次 trial；
  2. 每个 trial 的飞碟的色彩、大小、发射位置、速度、角度、同时出现的个数都可能不同。它们由该 round 的 ruler 控制；
  3. 每个 trial 的飞碟有随机性，总体难度随 round 上升；
  4. 鼠标点中得分，得分规则按色彩、大小、速度不同计算，规则可自由设定。
- 游戏的要求：
  - 使用带缓存的工厂模式管理不同飞碟的生产与回收，该工厂必须是场景单实例的！具体实现见参考资源 Singleton 模板类
  - 近可能使用前面 MVC 结构实现人机交互与游戏模型分离


## 项目地址与演示视频

项目地址 ->[传送门?](https://github.com/Tifinity/Unity3DStudy-master/tree/master/%E9%A1%B9%E7%9B%AE%E4%BA%94%EF%BC%9A%E7%AE%80%E5%8D%95%E6%89%93%E9%A3%9E%E7%A2%9F)

视频连接 -> [传送门?](https://www.bilibili.com/video/av70636842/)
## 具体实现

动作管理的大部分代码延用上一次作业，需要实现的就只有一个飞碟的飞行动作。

- FlyActionManager

  飞碟的动作管理类，当场景控制器需要发射飞碟时就调用DiskFly使飞碟飞行。

  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class FlyActionManager : SSActionManager {
      public DiskFlyAction fly;  
      public FirstController scene_controller;           
  
      protected void Start() {
          scene_controller = (FirstController)SSDirector.GetInstance().CurrentScenceController;
          scene_controller.action_manager = this;     
      }
  
      //飞碟飞行
      public void DiskFly(GameObject disk, float angle, float power) {
          int lor = 1;
          if (disk.transform.position.x > 0) lor = -1;
          fly = DiskFlyAction.GetSSAction(lor, angle, power);
          this.RunAction(disk, fly, this);
      }
  }
  
  ```

- DiskFlyAction

  通过位置变换和角度变换模拟飞碟的飞行，也可以使用刚体组件（Rigidbody）实现。当飞碟的高度在摄像机观察范围之下时则动作停止。

  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class DiskFlyAction : SSAction {
      public float gravity = -5;                                 //向下的加速度
      private Vector3 start_vector;                              //初速度向量
      private Vector3 gravity_vector = Vector3.zero;             //加速度的向量，初始时为0
      private Vector3 current_angle = Vector3.zero;              //当前时间的欧拉角
      private float time;                                        //已经过去的时间
  
      private DiskFlyAction() { }
      public static DiskFlyAction GetSSAction(int lor, float angle, float power) {
          DiskFlyAction action = CreateInstance<DiskFlyAction>();
          if (lor == -1) {
              action.start_vector = Quaternion.Euler(new Vector3(0, 0, -angle)) * Vector3.left * power;
          }
          else {
              action.start_vector = Quaternion.Euler(new Vector3(0, 0, angle)) * Vector3.right * power;
          }
          return action;
      }
  
      public override void Update() {
          time += Time.fixedDeltaTime;
          gravity_vector.y = gravity * time;
  
          transform.position += (start_vector + gravity_vector) * Time.fixedDeltaTime;
          current_angle.z = Mathf.Atan((start_vector.y + gravity_vector.y) / start_vector.x) * Mathf.Rad2Deg;
          transform.eulerAngles = current_angle;
  
          //如果物体y坐标小于-10，动作就做完了
          if (this.transform.position.y < -10) {
              this.destroy = true;
              this.callback.SSActionEvent(this);      
          }
      }
  
      public override void Start() { }
  }
  
  ```

然后是游戏对象部分，本次游戏中的对象就只有飞碟，为了节约资源需要使用工厂模式对飞碟进行管理。

- Disk

  Disk中保留飞碟类型type，分数score，颜色color。然后原本还有一个控制缩放的`public Vector3 scale = new Vector3(1, 0.25f, 1)`，我认为没有实际用处，因为碰撞检测器的大小会随着拉伸到比飞碟更大。
  

下图为拉伸之后的飞碟：
 ![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-g29drGwn-1570591537310)(T:\TH\大三上\3D游戏设计\5与游戏世界交互\打飞碟\image\lashen.jpg)\]](https://img-blog.csdnimg.cn/20191009112634972.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

  ```c#
  public class Disk : MonoBehaviour {
      public int type = 1;
      public int score = 1;                               
      public Color color = Color.white;                   
  }
  ```

- DiskFactory

  维护两个列表，一个是正在使用的飞碟，一个是空闲飞碟。当场景控制器需要获取一个飞碟时，先在空闲列表中寻找可用的空闲飞碟，如果找不到就根据预制重新实例化一个飞碟。回收飞碟的逻辑为遍历使用列表，当有飞碟已经完成了所有动作，即位置在摄像机之下，则回收。

  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class DiskFactory : MonoBehaviour {
      private List<Disk> used = new List<Disk>();
      private List<Disk> free = new List<Disk>();
  
      public GameObject GetDisk(int round) {
          GameObject disk_prefab;
          //寻找空闲飞碟,如果无空闲飞碟则重新实例化飞碟
          if (free.Count>0) {
              disk_prefab = free[0].gameObject;
              free.Remove(free[0]);
          } else {
              disk_prefab = Instantiate(
                  Resources.Load<GameObject>("Prefabs/disk"), 
                  new Vector3(0, -10f, 0), Quaternion.identity);
  
              disk_prefab.GetComponent<Renderer>().material.color = disk_prefab.GetComponent<Disk>().color;
              disk_prefab.transform.localScale = disk_prefab.GetComponent<Disk>().scale;
          }
  
          used.Add(disk_prefab.GetComponent<Disk>());
          disk_prefab.SetActive(true);
          return disk_prefab;
      }
  
      public void FreeDisk() {
          for(int i=0; i<used.Count; i++) {
              if (used[i].gameObject.transform.position.y <= -10f) {
                  free.Add(used[i]);
                  used.Remove(used[i]);
              }
          }          
      }
  
      public void Reset() {
          FreeDisk();
      }
  }
  ```

- UserGUI

  比较简单，将分数，Round，Trial显示出来，有按钮控制游戏开始和重新开始即可。比较重要的一点时使用`Input.GetButtonDown("Fire1")`检测鼠标左键的点击。

  ```c#
   if (Input.GetButtonDown("Fire1")) {
       Vector3 pos = Input.mousePosition;
       action.Hit(pos);
   }
  ```

  

- FirstController

  最重要的部分是场景控制器，游戏开始之后，设置一个定时器，每隔一定时间从飞碟工厂中获取一个飞碟并发射，检测用户点击发送的射线是否与飞碟发生碰撞，有则通知记分员加分并且通知工厂回收飞碟。部分重要函数代码如下。

  Update函数每一帧检测鼠标点击，并根据round调整规则。

  ```c#
  void Update () {
      if(running) {
          count++;
          //检测鼠标点击
          if (Input.GetButtonDown("Fire1")) {
              //Debug.Log("sdfsdfdf");
              Vector3 pos = Input.mousePosition;
              Hit(pos);
          }
          //ruler
          switch (round) {
              case 1: {
                  if (count >= 150) {
                      count = 0;
                      SendDisk(1);
                      trial += 1;
                      if (trial == 10) {
                          round += 1;
                          trial = 0;
                      }
                  }
                  break;
              }
              case 2: {
                  if (count >= 100) {
                      count = 0;
                      if (trial % 2 == 0) SendDisk(1);
                      else SendDisk(2);
                      trial += 1;
                      if (trial == 10) {
                          round += 1;
                          trial = 0;
                      }
                  }
                  break;
              }
              case 3: {
                  if (count >= 50) {
                      count = 0;
                      if (trial % 3 == 0) SendDisk(1);
                      else if(trial % 3 == 1) SendDisk(2);
                      else SendDisk(3);
                      trial += 1;
                      if (trial == 10) {
                          running = false;
                      }
                  }
                  break;
              }
              default:break;
          } 
          disk_factory.FreeDisk();
      }
  }
  ```

  SendDisk从工厂中拿飞碟并根据种类设置发射参数，然后调用动作管理器执行动作。师兄博客中提及的点一下会执行两次加分是因为将检测鼠标点击，即`Input.GetButtonDown("Fire1")`在UserGUI的OnGUI中实现，OnGUI再通知场景控制器执行Hit，调试了很久发现点一下调用了两次OnGUI，所以调用了两次Hit。并且我认为检测碰撞不应该由GUI实现所以在场景控制器中检测。

  ```c#
  private void SendDisk(int type) {
      //从工厂中拿一个飞碟
      GameObject disk = disk_factory.GetDisk(type);
      //飞碟位置
      float ran_y = 0;
      float ran_x = Random.Range(-1f, 1f) < 0 ? -1 : 1;
      //飞碟初始所受的力和角度
      float power = 0;
      float angle = 0;
      //根据飞碟种类不同设置不同的发射位置和速度
      if (type == 1) {
          ran_y = Random.Range(1f, 5f);
          power = Random.Range(5f, 7f);
          angle = Random.Range(25f,30f);
      }
      else if (type == 2) {
          ran_y = Random.Range(2f, 3f);
          power = Random.Range(10f, 12f);
          angle = Random.Range(15f, 17f);
      }
      else {
          ran_y = Random.Range(5f, 6f);
          power = Random.Range(15f, 20f);
          angle = Random.Range(10f, 12f);
      }
      disk.transform.position = new Vector3(ran_x*16f, ran_y, 0);
      action_manager.DiskFly(disk, angle, power);
  }
  ```

  Hit函数检测射线与飞碟是否碰撞，如碰撞则计分并回收飞碟。

  ```c#
  public void Hit(Vector3 pos) {
      Ray ray = Camera.main.ScreenPointToRay(pos);
      RaycastHit[] hits;
      hits = Physics.RaycastAll(ray);
      for (int i = 0; i < hits.Length; i++) {
          RaycastHit hit = hits[i];
          if (hit.collider.gameObject.GetComponent<Disk>() != null) {
              score_recorder.Record(hit.collider.gameObject);
              hit.collider.gameObject.transform.position = new Vector3(0, -10, 0);
          }
      }
  }
  ```

## 总结

本次作业主要使用了工厂模式复用已经实例化的对象节约资源，以及鼠标点击事件的检测，射线与游戏对象的碰撞。

## 参考资料

[师兄博客1](https://segmentfault.com/a/1190000014431406)

[师兄博客2](https://blog.csdn.net/c486c/article/details/79952255)

[官方文档](https://docs.unity3d.com/ScriptReference/Input.GetButtonDown.html)