---
title: "Unity3D项目三：牧师与魔鬼"
date: 2020-04-28T01:22:03+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# Unity3D项目三：牧师与魔鬼

### 基本介绍

- 列出游戏中提及的事物（Objects）

  牧师，恶魔，船，河流，左侧陆地，右侧陆地

- 用表格列出玩家动作表（规则表），注意，动作越少越好

  | 动作                   | 条件                                             | 结果           |
  | ---------------------- | ------------------------------------------------ | -------------- |
  | 点击角色（牧师或魔鬼） | 游戏未结束，角色在船上                           | 角色上岸       |
  | 点击角色（牧师或魔鬼） | 游戏未结束，角色在岸上，点击的角色与船在同一岸边 | 角色上船       |
  | 点击船                 | 游戏未结束，船上有至少一个角色                   | 船移动到另一侧 |
  | 点击重新开始           | 无                                               | 重新开始       |

- 请将游戏中对象做成预制

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921190059949.jpg)

- 在 GenGameObjects 中创建 长方形、正方形、球 及其色彩代表游戏中的对象。
- 使用 C# 集合类型 有效组织对象
- 整个游戏仅 主摄像机 和 一个 Empty 对象， **其他对象必须代码动态生成！！！** 。 整个游戏不许出现 Find 游戏对象， SendMessage 这类突破程序结构的 通讯耦合 语句。 **违背本条准则，不给分**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921185905453.jpg)
- 请使用课件架构图编程，**不接受非 MVC 结构程序**
- 注意细节，例如：船未靠岸，牧师与魔鬼上下船运动中，均不能接受用户事件！

### 完成过程

1. 首先用各种方块搭建好场景，主要是记住大概的位置，写代码的时候方便。

2. 基本架构 MVC

   - Model：游戏中所有GameObject，他们受各自Controllor的控制，我将这些对象控制器写在一个命名空间中方便调用。需要注意，Model并不是代码中的类，而是在游戏对象树中存在的实体。
   - View：GUI和Click，GUI负责展示游戏结果等信息，Click负责响应用户的点击。

   - Controllor：除了控制游戏物体的底层控制器，他们之上负责这些物体通讯加载以及根据用户输入执行不同动作的控制器是FirstControllor（场景控制器），场景控制器控制整个场景中所有游戏物体。在场景控制器之上为SSDirect（导演），本次游戏因为只有一个场景，所以导演的作用只负责退出和暂停。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921185828358.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)
3. 具体实现：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921185837855.jpg)

   - SSDirector：使用单例模式创建，并且继承于`System.Object`保持一直存在，不受Unity内存管理。导演负责控制当前场景的运行，切换，游戏的暂停恢复和退出，设定配置等等。

     ```c#
     //导演
     public class SSDirector : System.Object {
         private static SSDirector _instance;
         public ISceneController CurrentScenceController { get; set; }
         public static SSDirector GetInstance() {
             if (_instance == null) {
                 _instance = new SSDirector();
             }
             return _instance;
         }
     }
     ```

   - ISceneControllor：场景控制器的接口，导演使用这个接口与场景控制器沟通，本次作业中只有加载资源这一个函数需要实现。

     ```c#
     /*in class SSDirector*/
     public ISceneController CurrentScenceController { get; set; }
     
     /*in interface ISceneController*/
     public interface ISceneController {
         void LoadResources();
     }
     ```

   - IUserAction：用户操作的接口，根据上文中的用户动作表格缩写，使View与Controllor的桥梁。

     ```c#
     public interface IUserAction {
         //移动船
         void MoveBoat();
         //移动角色
         void MoveRole(RoleModel role);
         //重新开始
         void Restart();
         //检查游戏状态
         int Check();
     }
     ```

   - FirstControllor：场景控制器，是该场景中所有游戏对象“动起来”的核心，使用底层控制器控制各个对象，实现上文中的两个接口供GUI和导演使用。

     ```c#
     public class FirstControllor : MonoBehaviour, ISceneController, IUserAction {
         
         /*场景中的资源*/
         
         void Start() {
             SSDirector director = SSDirector.GetInstance();
             director.CurrentScenceController = this;
             GUI = gameObject.AddComponent<UserGUI>() as UserGUI;
             LoadResources();
         }
     
         public void LoadResources() {
             /*加载资源*/
         }
     ```

   - UserGUI：建立按钮和标签，不做赘述。

   - LandModel：陆地控制器，首先陆地有两块，所以需要标志是开始岸还是结束岸；而且陆地可以看作一个‘容器’，上面装了牧师和魔鬼（或许还可以装船），所以它有一系列空位置，用来放置角色。

     ```c#
     public class LandModel {
             GameObject land;
             //保存每个角色放在陆地上的位置
             Vector3[] positions;  
             //表示是开始岸还是结束岸
             int land_sign;   
             //该陆地上的角色
             RoleModel[] roles = new RoleModel[6];
             public LandModel(string land_mark) {
                 positions = new Vector3[] {
                     new Vector3(6.35F,2.25F,0), new Vector3(7.35f,2.25F,0), new Vector3(8.35f,2.25F,0),
                     new Vector3(9.35f,2.25F,0), new Vector3(10.35f,2.25F,0), new Vector3(11.35f,2.25F,0)
                 };
                 if (land_mark == "start") land_sign = 1;
                 else land_sign = -1;
                 land = Object.Instantiate(
                         Resources.Load<GameObject>("Prefabs/Land"),
                         new Vector3(9 * land_sign, 1, 0), Quaternion.identity) as GameObject;
                 land.name = land_mark + "Land";
             }
         
            /*many method*/
            
         }
     ```

   - BoatModel：船控制器，需要的属性为在哪一岸的标志，和在两个岸边的载人位置，船上的角色。船和陆地比较相似，只不过多了一个移动的动作。此处有一个需要注意的点是空位置的设置，startPos和endPos的x轴绝对值是要反过来的否则会出现这样的情况：

     两个角色上船，移动船，离岸远的角色A下船再上船，会与角色B重叠。

     ```c#
     startPos = new Vector3[] { new Vector3(5.5f, 1, 0), new Vector3(4.5f, 1, 0) };
     endPos = new Vector3[] { new Vector3(-4.5f, 1, 0), new Vector3(-5.5f, 1, 0) };
     ```

   - RoleModel：角色控制器，属性应有：牧师或恶魔的标志，还有在哪一块陆地和在船上的标志，方法应有移动到指定位置。而且我认为角色不能有船或者陆地，这样会使类之间耦合。

   - Move：控制移动，通过标志来区分如何移动，是先垂直再水平，还是先水平再垂直，或者只水平移动。使用Update来实现对象的运动。在使用控制器创建对象时会将该脚本挂载到需要运动的对象上。

     ```c#
     void Update() {
         if (move_sign == 1) {
             transform.position = Vector3.MoveTowards(
                 transform.position, middle_pos, move_speed * Time.deltaTime);
             if (transform.position == middle_pos)
                 move_sign = 2;
         }
         else if (move_sign == 2) {
             transform.position = Vector3.MoveTowards(
                 transform.position, end_pos, move_speed * Time.deltaTime);
             if (transform.position == end_pos)
                 move_sign = 0;
         }
     }
     ```

   - Click：检测鼠标点击，需要注意的是对象需要挂载Collider组件才会响应点击，不过创建的正方体等是默认挂了的，如果你用了其他的模型需要自己添加合适的碰撞检测器。在使用控制器创建对象时会将该脚本挂载到需要响应点击的对象上。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921185850133.jpg)
     ```c#
     public class Click : MonoBehaviour {
         IUserAction action;
         RoleModel role = null;
         BoatModel boat = null;
         public void SetRole(RoleModel role) {
             this.role = role;
         }
         public void SetBoat(BoatModel boat) {
             this.boat = boat;
         }
         void Start() {
             action = SSDirector.GetInstance().CurrentScenceController as IUserAction;
         }
         void OnMouseDown() {
             if (boat == null && role == null) return;
             if (boat != null) action.MoveBoat();
             else if (role != null) action.MoveRole(role);
         }
     }
     ```

     最后，将FirstCortrollor挂载到Main对象上即可运行。代码部分还是参考了师兄的，不过也发现了师兄的几个BUG并改正，而且将角色类与陆地和船的耦合性降低，对代码的层次也重新构建了一下。资源都是商店里免费筛选的前几个，搜Devil和Boat就能找到，牧师的话在免费的前几页。

4. 关于动画

   如果你下载的资源自带动画，选中动画的文件在检查器中点击Edit，勾上循环时间和循环动作，再将这个文件像贴图一样拖到对象组件的动画器中即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921185752190.jpg)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921185800962.jpg)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20190921185808336.jpg)
演示视频：[?](https://www.bilibili.com/video/av68513475/)

github项目传送门：[?](https://github.com/Tifinity/Unity3DStudy-master/tree/master/项目三：牧师与魔鬼)

感谢师兄的博客参考：[?](https://blog.csdn.net/C486C/article/details/79795708)