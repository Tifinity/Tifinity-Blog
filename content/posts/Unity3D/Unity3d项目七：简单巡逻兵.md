---
title: "Unity3d项目七：简单巡逻兵"
date: 2020-04-28T00:16:44+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# 简单巡逻兵

@[toc]
## 游戏规则与要求

- 智能巡逻兵
  - 游戏设计要求：
    - 创建一个地图和若干巡逻兵(使用动画)；
    - 每个巡逻兵走一个3~5个边的凸多边型，位置数据是相对地址。即每次确定下一个目标位置，用自己当前位置为原点计算；
    - 巡逻兵碰撞到障碍物，则会自动选下一个点为目标；
    - 巡逻兵在设定范围内感知到玩家，会自动追击玩家；
    - 失去玩家目标后，继续巡逻；
    - 计分：玩家每次甩掉一个巡逻兵计一分，与巡逻兵碰撞游戏结束；
  - 程序设计要求：
    - 必须使用订阅与发布模式传消息
      - subject：OnLostGoal
      - Publisher: GameEventManager
      - Subscriber: 场景控制器
    - 工厂模式生产巡逻兵

## 项目地址与演示视频

项目地址 -> [传送门?](https://github.com/Tifinity/Unity3DStudy-master/tree/master/%E9%A1%B9%E7%9B%AE%E4%B8%83%EF%BC%9A%E7%AE%80%E5%8D%95%E5%B7%A1%E9%80%BB%E5%85%B5)

视频连接 -> [传送门?]( https://www.bilibili.com/video/av73465390/ )

### Unity3D小技巧：

- 窗口是可以锁定的：

  当你制作预制体需要把脚本拖到检查器中时，按一下窗口右上角的锁就可以锁定窗口，选中其他对象检查器也不会变，告别手残·~·

- 去掉轮廓：

  有时布置场景时感觉对象图标和轮廓太碍事，去掉他们！

  Gizmos->调整图标大小，显示轮廓网格选择线。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026160436747.jpg)

## 具体实现

基本逻辑：玩家通过键盘控制人物在地图上移动，地图分为九个部分，有八个巡逻兵在出生点以外的部分巡逻，当玩家进入一个部分时该部分的巡逻兵就会追逐玩家，若玩家甩开巡逻兵则分数加一，若玩家与巡逻兵触碰则游戏结束。

由于本次项目代码量比较大，框架部分还是重用之前写的代码，在此只叙述重点部分。

本章标题是动画与模型，于是我将重点放在了人物模型与动画上。

我所使用的模型素材：[?](https://share.weiyun.com/5vZ22uQ)

### 人物模型部分：

由于这部分内容比较多放在了另一篇博客中：[?](https://blog.csdn.net/Tifinity/article/details/102762493)

人物模型如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026154042982.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

人物动画器如下，一个Layer五个动作：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026154146745.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)



### 巡逻兵部分：

巡逻兵与玩家使用同一个模型和动画器，不过通过代码控制其只有行走和奔跑两个动作，巡逻时行走，追逐玩家时奔跑。

![在这里插入图片描述](https://img-blog.csdnimg.cn/201910261542005.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

巡逻兵的移动策略是参考[C486C师兄(师姐？)博客]( https://blog.csdn.net/c486c/article/details/80153548 )的，不过师兄所有的动作都是使用运动学的函数MoveTowards来实现并且使用Update来更新，而巡逻兵是刚体且没有勾选运动学，我认为使用运动学的函数来移动刚体是不太好的，所以改为物理引擎的移动方式并且使用FixedUpdate来更新；另一点是师兄将本次巡逻兵的两个动作切换写在了动作管理器基类SSActionManager中，我认为应该在巡逻兵的动作管理器中GuardActionManager中实现该功能。

以下分析部分代码：

- GuardPatrolAction：

  Update只更新平面移动的向量，FixedUpdate使用该向量进行移动，通过修改rigid的velocity来移动。

  ~~~C#
  /*变量*/
  private Rigidbody rigid;    //刚体组件
  private Vector3 planarVec;  // 平面移动向量
  
  /*更新函数*/
  public override void Update() {
      //保留供物理引擎调用
      planarVec = gameobject.transform.forward * data.walkSpeed;
  }
  
  public override void FixedUpdate() {
      //巡逻
      Gopatrol();
      //玩家进入该区域，巡逻结束，开始追逐
      if (data.playerSign == data.sign) {
          this.destroy = true;
          this.callback.SSActionEvent(this, SSActionEventType.Competeted, 0, this.gameobject);
      }
  }
  
  /*in Gopatrol()*/
  rigid.velocity = new Vector3(planarVec.x, rigid.velocity.y, planarVec.z);
  ~~~

- GuardFollowAction:

  与GuardPatrolAction相同，改为物理引擎的方式。

  ~~~c#
   public override void Update() {
       //保留供物理引擎调用
       planarVec = gameobject.transform.forward * speed;
   }
  
  public override void FixedUpdate() {
      transform.LookAt(player.transform.position);
      rigid.velocity = new Vector3(planarVec.x, rigid.velocity.y, planarVec.z);
  
      //如果玩家脱离该区域则继续巡逻
      if (data.playerSign != data.sign) {
          this.destroy = true;
          this.callback.SSActionEvent(this, SSActionEventType.Competeted, 1, this.gameobject);
      }
  }
  ~~~

- GuardActionManager：

  主要实现巡逻兵动作切换的功能，实现回调函数SSActionEvent，当一个动作被销毁时调用另一个动作。

  ~~~c#
  public void SSActionEvent(
      SSAction source, SSActionEventType events = SSActionEventType.Competeted,
      int intParam = 0, GameObject objectParam = null) {
      if (intParam == 0) {
          //追逐
          GuardFollowAction follow = GuardFollowAction.GetSSAction(player);
          this.RunAction(objectParam, follow, this);
      } else {
          //巡逻
          GuardPatrolAction move = GuardPatrolAction.GetSSAction(objectParam.gameObject.GetComponent<GuardData>().start_position);
          this.RunAction(objectParam, move, this);
          Singleton<GameEventManager>.Instance.PlayerEscape();
      }
  }
  ~~~




### 地图部分：

地图分为九个格子，每个区域有自己的碰撞器，并挂载AreaCollide.cs。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026154214488.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026154222336.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)



- AreaCollide：

  当玩家进入时设置场景控制器中的玩家区域标志，然后场景控制器通知对应的巡逻兵追逐玩家。

~~~C#
public class AreaCollide : MonoBehaviour {
    public int sign = 0;
    private FirstSceneController sceneController;

    private void Start() {
        sceneController = SSDirector.GetInstance().CurrentScenceController as FirstSceneController;
    }
	
    void OnTriggerEnter(Collider collider) {
        if (collider.gameObject.tag == "Player") {
            sceneController.playerSign = sign;
        }
    }
}
~~~



### 观察者模式：
- GameEventManager发布消息：

  ~~~c#
  public class GameEventManager : MonoBehaviour {
      //定义回调函数类型和subject
      public delegate void ScoreEvent();
      public static event ScoreEvent ScoreChange;
      
      public delegate void GameoverEvent();
      public static event GameoverEvent GameoverChange;
  	
      public void PlayerEscape() {
          if (ScoreChange != null) {
              ScoreChange();
          }
      }
  
      public void PlayerGameover(){
          if (GameoverChange != null) {
              GameoverChange();
          }
      }
  }
  ~~~

- FirstSenceController类订阅消息：

  ~~~c#
  void OnEnable() {
      GameEventManager.ScoreChange += AddScore;
      GameEventManager.GameoverChange += Gameover;
  }
  void OnDisable() {
      GameEventManager.ScoreChange -= AddScore;
      GameEventManager.GameoverChange -= Gameover;
  }
  
  void AddScore() {
      recorder.AddScore();
  }
  
  void Gameover() {
      game_over = true;
  }
  ~~~

  




## 总结

本次作业主要是了解了物理引擎在Unity3D中的使用，包括FixedUpdate函数，刚体组件，对游戏对象速度和力的操作，使用物理引擎就不需要模拟运动学的位置变换，使用复杂的公式计算。总的来说过程比较顺利，没有太大的阻碍。



## 参考资料

[优秀博客 C486C](https://blog.csdn.net/c486c/article/details/80153548#%E7%AE%80%E5%8D%95%E5%B7%A1%E9%80%BB%E5%85%B5)