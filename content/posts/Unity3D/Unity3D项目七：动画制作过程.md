---
title: "Unity3D项目七：动画制作过程"
date: 2020-04-28T00:15:46+08:00
draft: false
categories: ["Unity3D"]
tags: ["none"]
featured_image: 
author: "Tifinity"
---

# 人物模型与动画

@[toc]
## 效果展示

作业博客 -> [传送门]( https://blog.csdn.net/Tifinity/article/details/102757393 )

视频连接 -> [传送门]( https://www.bilibili.com/video/av73465390/ )



## 具体实现

### 准备工作

- 素材资源

  我所使用的模型素材：[传送门](https://share.weiyun.com/5vZ22uQ)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231127243.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231139621.jpg)

		从左到右从上到下分别为行走，奔跑，跳跃，后跳，翻滚，下落，（还有一个站立忘记截图）这就是本次使用的所有动画。

- 制作预制体

  预制体结构树：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019102623114823.jpg)

  首先创建Player空对象，其子对象为：ybot人物模型，cameraHandle空对象用于控制相机，sensor空对象用于探测地面。

### 动画器

- 准备工作

  在Assert中创建新文件夹Animator->创建动画控制器，命名为PlayerController，

  将所使用的动画都拖入人物动画器的Base Layer中。

  最后动画器长这个样子：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231201181.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

  实现的效果是，按下WASD进行行走和转向，站立不动时按下空格后跳一小步，行走时按下空格向前翻滚，按住SHIFT奔跑，奔跑时按下空格向前跳跃，在空中跳跃完成后下落，下落到地面时翻滚。

  然后开始一步一步制作。

  

- ground动画混合树

  ground是一个Blend Tree，动画混合树，一个动画混合树其实也是一个状态，不过可以由多个动画所组成。

  [官网动画混合树教程]( https://docs.unity3d.com/Manual/class-BlendTree.html )

  在BaseLayer右键->创建状态->从新的混合树，创建新的动画混合树。

  ground由站立，走路，奔跑三个动画组成，将这三个动画拖进来。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231249753.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)


  修改混合树名字和参数名字，在Motion中新增三个状态，并拖入如图所示的三个动画，并调整阈值。拖动红色标尺能看到动画的渐变。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231301422.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpZmluaXR5,size_16,color_FFFFFF,t_70)

  

- 创建过渡与参数控制

  右键->创建过渡，将所有状态都连接起来。状态之间的过渡通过参数来控制。

  使用的所有参数如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231316607.jpg)

  forward为Float，用于控制ground混合树中行走奔跑的过渡；

  OnGround为Bool，表示模型是否在地面上；

  jump为Trigger，用于控制跳跃和后跳；

  roll为Trigger，用于控制翻滚；

  jabVelocity为Float，表示后跳的速度；

  rollVelocity为Float，表示翻滚的速度。

  

  下面是一个从ground到jump的过渡例子：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231326161.jpg)

  使用参数来过渡需要不勾选退出时间，两个动画的长短以及过渡的区间都可以调整，在预览中看到自己满意的效果即可。

  

  然后是另一个例子：

  从jump到fall会有一个问题，如果跳起来还没有播放下落动画就着地了，此时会接着播放fall再播放ground，我们就需要将这条路中断。在状态的Setting中选取中断源为Current State，CurrentState就是过渡箭头的起始状态，在这里就是强行中断fall返回jump再回到ground。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231336640.jpg)

  其他过渡不再赘述。

  

- 通过脚本控制参数

  创建ActorController.cs，一些实现的细节在代码与注释中：

  ~~~c#
  public GameObject model;    		//人物模型
  public PlayerInput pi;				//用户输入
  public float walkSpeed = 1.5f;		//行走速度
  public float runMultiplier = 2.7f;	//奔跑速度
  public float jumpVelocity = 4f;		//跳跃速度
  public float rollVelocity = 1f;		//翻滚速度
  
  [SerializeField]
  private Animator anim;				//动画控制器
  private Rigidbody rigid;			//刚体组件
  private Vector3 planarVec; 			//平面移动向量
  private Vector3 thrustVec;			//跳跃冲量
  
  private bool lockPlanar = false;    //跳跃时锁死平面移动向量
  
  void Awake() {
      pi = GetComponent<PlayerInput>();
      anim = model.GetComponent<Animator>();
      rigid = GetComponent<Rigidbody>();
  }
  ~~~

  

  Update()

  ~~~C#
  //刷新每秒60次
  private void Update() {
      //修改动画混合树
      /*1.从走路到跑步没有过渡*/
      /*anim.SetFloat("forward", pi.Dmag * (pi.run ? 2.0f : 1.0f));*/
      /*2.使用Lerp加权平均解决*/
      float targetRunMulti = pi.run ? 2.0f : 1.0f;
      anim.SetFloat("forward", pi.Dmag * Mathf.Lerp(anim.GetFloat("forward"), targetRunMulti, 0.3f));
  
      //播放翻滚动画
      if (rigid.velocity.magnitude > 1.0f) {
          anim.SetTrigger("roll");
      }
      //播放跳跃动画
      if (pi.jump) {
          anim.SetTrigger("jump");
      }
  
      //转向
      if(pi.Dmag > 0.01f) {
          /*1.旋转太快没有补帧*/
          /*model.transform.forward = pi.Dvec;*/
          /*2.使用Slerp内插值解决*/
          Vector3 targetForward = Vector3.Slerp(model.transform.forward, pi.Dvec, 0.2f);
          model.transform.forward = targetForward;
      }
  
      if(!lockPlanar) {
          //保存供物理引擎使用
          planarVec = pi.Dmag * model.transform.forward * walkSpeed * (pi.run ? runMultiplier : 1.0f);
      }
  }
  ~~~

  FixedUpdate()

  ~~~c#
  //物理引擎每秒50次
  private void FixedUpdate() {
      //Time.fixedDeltaTime 50/s
      //1.修改位置
      //rigid.position += movingVec * Time.fixedDeltaTime;
      //2.修改速度
      rigid.velocity = new Vector3(planarVec.x, rigid.velocity.y, planarVec.z) + thrustVec;
      //仅一帧，跳跃冲量
      thrustVec = Vector3.zero;
  }
  ~~~

  

- 获取玩家输入

  创建PlayerInput.cs，平滑地控制角色的行走与转向，在空中不允许用户输入，锁死移动向量，同时还需要解决同时按下W和A时移动速度会与只按下W不同的问题，即将矩形坐标转为圆坐标。在ActorController.cs中使用获取的输入。

  ~~~c#
  public class PlayerInput : MonoBehaviour {
      [Header("---- KeyCode Settings ----")]
      /*方向键*/
      public string keyUp = "w";
      public string keyDown = "s";
      public string keyLeft = "a";
      public string keyRight = "d";
  	/*功能键*/
      public string keyA = "left shift";
      public string keyB = "space";
      public string keyC = "k";
      public string keyD;
  	/*镜头控制*/
      public string keyJUp = "up";
      public string keyJDown = "down";
      public string keyJLeft = "left";
      public string keyJRight = "right";
  
      [Header("---- Output Settings ----")]
      /*角色方向*/
      public float Dup;		
      public float Dright;
      public float Dmag;
      /*角色速度*/
      public Vector3 Dvec;
      /*镜头方向*/
      public float Jup;		
      public float Jright;
      /*奔跑跳跃参数*/
      public bool run;		
      public bool jump;
      private bool lastJump;
  
      [Header("---- Other Settings ----")]
      public bool inputEnabled = true;		//是否允许输入  	
      private float targetDup;
      private float targetDright;
      private float velocityDup;
      private float velocityDright;
  
      void Start() {}
  
      void Update() {
          Jup = (Input.GetKey(keyJUp)) ? 1.0f : 0 - (Input.GetKey(keyJDown) ? 1.0f : 0);
          Jright = (Input.GetKey(keyJRight)) ? 1.0f : 0 - (Input.GetKey(keyJLeft) ? 1.0f : 0);
  
          targetDup = (Input.GetKey(keyUp) ? 1.0f : 0) - (Input.GetKey(keyDown) ? 1.0f : 0);
          targetDright = (Input.GetKey(keyRight) ? 1.0f : 0) - (Input.GetKey(keyLeft) ? 1.0f : 0);
  
          if(!inputEnabled) {
              targetDup = 0;
              targetDright = 0;
          }
          /*平滑变动*/
          Dup = Mathf.SmoothDamp(Dup, targetDup, ref velocityDup, 0.1f);
          Dright = Mathf.SmoothDamp(Dright, targetDright, ref velocityDright, 0.1f);
  
          /*矩形坐标转圆坐标*/
          Vector2 tempDAxis = SquareToCircle(new Vector2(Dup, Dright));
          float Dup2 = tempDAxis.x;
          float Dright2 = tempDAxis.y;
  
          Dmag = Mathf.Sqrt((Dup2 * Dup2) + (Dright2 * Dright2));
          Dvec = Dright * transform.right + Dup * transform.forward;
          run = Input.GetKey(keyA);
  
          /*跳跃*/
          bool newJump = Input.GetKey(keyB);
          lastJump = jump;
          if(lastJump == false && newJump == true) {
              jump = true;
          }
          else {
              jump = false;
          }
      }
  
      /*矩形坐标转圆坐标*/
      private Vector2 SquareToCircle(Vector2 input) {
          Vector2 output = Vector2.zero;
          output.x = input.x * Mathf.Sqrt(1 - (input.y * input.y) / 2.0f);
          output.y = input.y * Mathf.Sqrt(1 - (input.x * input.x) / 2.0f);
          return output;
      }
  }
  ~~~

- FSM

  接下来创建一些用于发送消息的脚本。

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231350923.jpg)

  以FSMOnEnter.cs为例，进入状态时向父级发送消息。

  ~~~C#
  public class FSMOnEnter : StateMachineBehaviour {
      public string[] onEnterMessages;
  
      override public void OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex) {
          foreach (var msg in onEnterMessages) {
              animator.gameObject.SendMessageUpwards(msg);
          }
      }
  }
  ~~~

  还是以jump为例，在jump状态中添加行为->FSMOnEnter，然后将size设为1，手动输入OnJumpEnter，当进入jump状态时动画器会向父级发出OnJumpEnter信息。

  然后在ActorController中加上如下函数来接受消息。

  ~~~C#
  public void OnJumpEnter() {
      pi.inputEnabled = false;
      lockPlanar = true;
      thrustVec = new Vector3(0, jumpVelocity, 0);
  }
  ~~~

  接受进入跳跃的消息，锁定跳跃时平面移动向量，并且给人物模型一个冲量来跳跃。

  FSMOnExit在状态退出时发出消息；

  FSMOnUpdate在状态刷新时发出消息，用来实现翻滚和跳跃；

  FSMClearSignals比较特殊，在状态进入和退出时清除多余的Trigger，否则按一下空格会给jump参数储存多个Trigger，跳跃多次。

  

- 实现后跳和翻滚的速度匹配

  后跳和翻滚时，人物模型的速度需要进行调整，以匹配动画的播放速度。

  ~~~c#
  public void OnJabUpdate() {
      thrustVec = model.transform.forward * anim.GetFloat("jabVelocity") * 1.4f;
  }
  
  public void OnRollUpdate() {
      thrustVec = model.transform.forward * anim.GetFloat("rollVelocity") * 1.0f;
  }
  ~~~

  选中jump2->Edit：

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026231421707.jpg)

  后跳动画需要勾选上根变换位置（Y），这样跳的时候模型才会有Y轴上的位置变化。

  然后在曲线中，增加新的曲线，与jabVelocity参数同名，双击进入增加新的Key，设置为-3，最后记得一定要点应用。这样在后跳的时候jabVelocity参数会随该曲线变化，然后在OnJabUpdate中调用即可。

  翻滚动作也是同样的操作。



### 实现第三人称

在cameraHandle中增加一个空对象cameraPos位置调整到人物模型的脖子后方，并挂载CameraController.cs脚本，用于控制摄像机的位置，实现第三人称。详细的就不再赘述了，看代码即可。

~~~C#
/*因为人物是通过物理引擎来移动，摄像机也需要使用FixedUpdate*/
public class CameraConrtoller : MonoBehaviour {
    public PlayerInput pi;
    public float horizontalSpeed = 100f;		//水平移动速度
    public float verticalSpeed = 80f;			//垂直移动速度
    public float cameraDampValue = 0.5f;
    
    private GameObject playerHandle;
    private GameObject cameraHandle;
    private float tempEulerX;
    private GameObject model;
    private GameObject camera;

    private Vector3 cameraDampVelocity;
    
    void Awake() {
        cameraHandle = transform.parent.gameObject;
        playerHandle = cameraHandle.transform.parent.gameObject;
        model = playerHandle.GetComponent<ActorController>().model;
        camera = Camera.main.gameObject;
        tempEulerX = 20f;
    }

    void FixedUpdate() {
        Vector3 tempModelEuler = model.transform.eulerAngles;
        playerHandle.transform.Rotate(Vector3.up, pi.Jright * horizontalSpeed * Time.fixedDeltaTime);
        tempEulerX -= pi.Jup * verticalSpeed * Time.fixedDeltaTime;
        tempEulerX = Mathf.Clamp(tempEulerX, -35, 30);
        cameraHandle.transform.localEulerAngles = new Vector3(tempEulerX, 0, 0);
        model.transform.eulerAngles = tempModelEuler;

        camera.transform.position = Vector3.SmoothDamp(
            camera.transform.position, transform.position, 
            ref cameraDampVelocity, cameraDampValue);
        camera.transform.eulerAngles = transform.eulerAngles;
    }
}
~~~