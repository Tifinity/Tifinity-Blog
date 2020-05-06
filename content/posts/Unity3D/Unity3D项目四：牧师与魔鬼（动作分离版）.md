---
title: "Unity3D项目四：牧师与魔鬼（动作分离版）"
date: 2020-04-28T00:24:09+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# Unity3D项目四：牧师与魔鬼（动作分离版）

### 基本介绍

动作管理是游戏的重要内容，全部都放在游戏对象里显得十分笨重，所以本次项目需要将动作从对象中提取出来写成单独的动作控制器。动作控制器来管理控制所有的游戏对象移动，通过场景控制器将需要移动的游戏对象和位置等信息传递给动作控制器，动作控制器负责实现具体的移动。当动作很多或是需要做同样动作的游戏对象很多的时候，使用动作管理器可以让动作很容易管理，也提高了代码复用性。

### 具体实现

#### 完成ActionControllor.cs

- ISSActionCallback（动作事件接口）

  使用了枚举变量定义动作类型，执行还是完成。

  定义了事件处理接口，动作组合类和动作管理类都需要实现这个接口，来实现接收其子动作的消息以实现动作的调度管理。当动作完成的时候会调用该接口通知管理者该动作完成。

  ```c#
  public enum SSActionEventType : int { Started, Competeted }
  
  public interface ISSActionCallback {
      void SSActionEvent(
          SSAction source, SSActionEventType events = SSActionEventType.Competeted,
          int intParam = 0, string strParam = null, Object objectParam = null
      );
  }
  ```

- SSAction（动作基类）

  代表一个动作，继承了ScriptableObject，是不需要绑定GameObject的可编程基类，受到Unity管理。

  ```c#
  /*动作基类*/
  public class SSAction : ScriptableObject {
      public bool enable = true;                      //是否进行
      public bool destroy = false;                    //是否删除
  
      public GameObject gameobject;                   //动作对象
      public Transform transform;                     //动作对象的transform
      public ISSActionCallback callback;              //回调函数
  
      //防止用户自己new对象
      protected SSAction() { }                      
  
      public virtual void Start() {
          throw new System.NotImplementedException();
      }
  
      public virtual void Update() {
          throw new System.NotImplementedException();
      }
  }
  ```

- SSMoveToAction（移动位置子类）

  继承动作基类，实现移动到指定位置的子类。

  ```c#
  /*子类 - 移动到指定位置*/
  public class SSMoveToAction : SSAction {
      public Vector3 target;  
      public float speed;
  
      public static SSMoveToAction GetSSAction(Vector3 _target, float _speed) {
          //让unity自己创建一个SSMoveToAction实例确保内存正确回收
          SSMoveToAction action = ScriptableObject.CreateInstance<SSMoveToAction>();
          action.target = _target;
          action.speed = _speed;
          return action;
      }
  
      //C#必须显示声明重写父类虚函数
      public override void Start() {
          //该动作建立时无需任何操作
      }
  
      public override void Update() {
          this.transform.position = Vector3.MoveTowards(this.transform.position, target, speed * Time.deltaTime);
          //动作完成，通知动作管理者或动作组合
          if (this.transform.position == target) {
              this.destroy = true;
              this.callback.SSActionEvent(this);      
          }
      }
  }
  ```

- SequenceAction（组合动作类）

  组合类中实现回调函数的接口ISSActionCallback，动作列表中每一个动作完成之后通知组合动作类，并且所有动作完成后也会通知该这个类的上一层组合类，被组合的对象和组合对象属于同一种类型，之后我们的动作也都是通过组合动作实现的，这样的结构稳固而可延展，可以实现我们所需要的绝大部分动作管理。

  ```c#
  /*动作组合*/
  public class SequenceAction : SSAction, ISSActionCallback {
      public List<SSAction> sequence;    //动作的列表
      public int repeat = -1;            //-1就是无限循环做组合中的动作
      public int start = 0;              //当前做的动作的索引
  
      public static SequenceAction GetSSAcition(int repeat, int start, List<SSAction> sequence) {
          //让unity自己创建一个SequenceAction实例
          SequenceAction action = ScriptableObject.CreateInstance<SequenceAction>();
          action.sequence = sequence;
          action.repeat = repeat;
          action.start = start;
          return action;
      }
  
      public override void Start() {
          //对每个动作执行
          foreach (SSAction action in sequence) {
              action.gameobject = this.gameobject; 
              action.transform = this.transform;
              action.callback = this;                 //每一个动作的回调函数为该动作组合
              action.Start();
          }
      }
  
      public override void Update() {
          if (sequence.Count == 0) return;
          if (start < sequence.Count) {
              sequence[start].Update();               //start在回调函数中递加
          }
      }
  
      //实现接口ISSActionCallback
      public void SSActionEvent(
          SSAction source, SSActionEventType events = SSActionEventType.Competeted,
          int intParam = 0, string strParam = null, Object objectParam = null
      ) {
          source.destroy = false;                     //可能会无限循环所以先不删除
          this.start++;                               //下一个动作
          if (this.start >= sequence.Count) {
              this.start = 0;
              if (repeat > 0) repeat--;               //repeat<0就不会再减小
              if (repeat == 0) {                      //动作组合结束
                  this.destroy = true;                //删除
                  this.callback.SSActionEvent(this);  //通知管理者
              }
          }
      }
  
      void OnDestroy() {
          //如果自己被删除则应该释放自己管理的动作   
      }
  }
  ```

- SSActionManger（动作管理基类）

  负责管理所有的动作或动作组合，给他们绑定游戏对象并且控制动作的执行或者完成后删除，也实现了ISSActionCallback接口，用于接受动作是否完成的消息，不过在我们的游戏之中对象仅有移动位置这个动作，所以回调函数不需要做任何事情。

  ```c#
  /*动作管理基类*/
  public class SSActionManager : MonoBehaviour, ISSActionCallback {
      private Dictionary<int, SSAction> actions = new Dictionary<int, SSAction>();    //动作字典
      private List<SSAction> waitingAdd = new List<SSAction>();                       //等待执行的动作列表
      private List<int> waitingDelete = new List<int>();                              //等待删除动作的key的列表                
  
      protected void Update() {
          //获取动作实例将等待执行的动作加入字典并清空待执行列表
          foreach (SSAction ac in waitingAdd) {
              actions[ac.GetInstanceID()] = ac;                                       
          }
          waitingAdd.Clear();
  
          //对于字典中每一个pair，看是执行还是删除
          foreach (KeyValuePair<int, SSAction> kv in actions) {
              SSAction ac = kv.Value;
              if (ac.destroy) {
                  waitingDelete.Add(ac.GetInstanceID());
              }
              else if (ac.enable) {
                  ac.Update();
              }
          }
  
          //删除所有已完成的动作并清空待删除列表
          foreach (int key in waitingDelete) {
              SSAction ac = actions[key];
              actions.Remove(key);
              Object.Destroy(ac);
          }
          waitingDelete.Clear();
      }
  
      public void RunAction(GameObject gameobject, SSAction action, ISSActionCallback manager) {
          action.gameobject = gameobject;
          action.transform = gameobject.transform;
          action.callback = manager;
          waitingAdd.Add(action);
          action.Start();
      }
  
      public void SSActionEvent(
          SSAction source, SSActionEventType events = SSActionEventType.Competeted,
          int intParam = 0, string strParam = null, Object objectParam = null) {
      }
  }
  ```

- PADSceneActionManager（牧师与魔鬼场景的动作管理类）

  先与对应的场景控制器绑定，然后实现两个具体动作：

  1. 船移动：一个动作，从当前位置以speed的速率移动到endPos。
  2. 角色移动：两个动作，从当前位置以speed的速率移动到middlePos，再以speed的速率移动到endPos。

  ```c#
  /*本次游戏场景中动作管理类*/
  public class PADSceneActionManager : SSActionManager {
      public FirstControllor sceneController;
  
      private SequenceAction boatMove;
      private SequenceAction roleMove;
  
      protected void Start() {
          sceneController = (FirstControllor)SSDirector.GetInstance().CurrentScenceController;
          sceneController.actionManager = this;
      }
  
      protected new void Update() {
          base.Update();
      }
  
      public void moveBoat(GameObject boat, Vector3 endPos, float speed) {
          SSAction action1 = SSMoveToAction.GetSSAction(endPos, speed);
          boatMove = SequenceAction.GetSSAcition(0, 0, new List<SSAction> { action1 });
          this.RunAction(boat, boatMove, this);
      }
  
      public void moveRole(GameObject role, Vector3 middlePos, Vector3 endPos, float speed) {
          //两段移动
          SSAction action1 = SSMoveToAction.GetSSAction(middlePos, speed);
          SSAction action2 = SSMoveToAction.GetSSAction(endPos, speed);
          //repeat=1，start=0，两个动作
          roleMove = SequenceAction.GetSSAcition(1, 0, new List<SSAction> { action1, action2 });
          this.RunAction(role, roleMove, this);
      }
  }
  ```

  至此动作管理的代码都已实现。

#### 修改已有代码

- 将MoveCortrollor.cs删除，有了动作管理类就不再需要移动控制器了。

- 将BoatModel和RoleModel中的与运动相关的方法删去，添加各自的speed。

- 在FirstCortrollor类中加入动作管理类

  `public PADSceneActionManager actionManager; //添加动作管理类`

  并且修改MoveBoat和MoveRole，使用动作管理器来实现，场景控制器只负责传递游戏对象和位置信息。

#### 新增裁判类

- 将原来的check从用户接口中删除，check的实现从FirstCortrollor.cs中删除

- 新增Check.cs

  ```c#
  using UnityEngine;
  
  namespace CheckApplication {
      public class Check : MonoBehaviour {
          public FirstControllor sceneController;
  
          protected void Start() {
              sceneController = (FirstControllor)SSDirector.GetInstance().CurrentScenceController;
              sceneController.gameStatusManager = this;
          }
  
          public int CheckGame() {
              //0-游戏继续，1-失败，2-成功
              int startPriests = (sceneController.startLand.GetRoleNum())[0];
              int startDevils = (sceneController.startLand.GetRoleNum())[1];
              int endPriests = (sceneController.endLand.GetRoleNum())[0];
              int endDevils = (sceneController.endLand.GetRoleNum())[1];
              if (endPriests + endDevils == 6) return 2;
              int[] boatNum = sceneController.boat.GetRoleNumber();
              //加上船上的人
              if (sceneController.boat.GetBoatSign() == 1) {
                  startPriests += boatNum[0];
                  startDevils += boatNum[1];
              }
              else {
                  endPriests += boatNum[0];
                  endDevils += boatNum[1];
              }
              if ((endPriests > 0 && endPriests < endDevils) || (startPriests > 0 && startPriests < startDevils)) {
                  return 1;
              }
              return 0;
          }
      }
  }
  ```

- 在FirstCortrollor中增加裁判：` public Check gameStatusManager;`





github项目地址->[?](https://github.com/Tifinity/Unity3DStudy-master/tree/master/%E9%A1%B9%E7%9B%AE%E5%9B%9B%EF%BC%9A%E7%89%A7%E5%B8%88%E4%B8%8E%E9%AD%94%E9%AC%BC%E5%8A%A8%E4%BD%9C%E5%88%86%E7%A6%BB%E7%89%88)

演示视频地址->[?](https://www.bilibili.com/video/av68513475/)

视频与原版无任何差别。

再次感谢师兄的博客供参考学习->[?](https://blog.csdn.net/C486C/article/details/79854679)