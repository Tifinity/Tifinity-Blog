---
title: "Unity3D项目十：简单坦克大战"
date: 2020-04-28T00:09:20+08:00
draft: false
categories: ["Unity3D"]
tags: ["Unity3D"]
featured_image: 
author: "Tifinity"
---

# 简单坦克大战
## 作业要求
坦克对战游戏 AI 设计
从商店下载游戏：“Kawaii” Tank 或 其他坦克模型，构建 AI 对战坦克。具体要求
- 使用“感知-思考-行为”模型，建模 AI 坦克
- 场景中要放置一些障碍阻挡对手视线
- 坦克需要放置一个矩阵包围盒触发器，以保证 AI 坦克能使用射线探测对手方位
- AI 坦克必须在有目标条件下使用导航，并能绕过障碍。（失去目标时策略自己思考）
- 实现人机对战

## 项目地址
[github](https://github.com/Tifinity/Unity3D-Learning/tree/master/%E9%A1%B9%E7%9B%AE%E4%B9%9D%EF%BC%9A%E7%AE%80%E5%8D%95%E5%9D%A6%E5%85%8B%E5%A4%A7%E6%88%98)
[bilibili](https://www.bilibili.com/video/av78119019/)
## 具体实现
参考[学姐博客](https://yuandi-sherry.github.io/2018/06/19/Week15-Tanks/)，对一些混乱的地方进行了修改。
关于框架的复用不再赘述，主要包括：
- 用工厂处理子弹和电脑坦克；
- 导演，场景控制器；
- IUserGUI接受用户输入；

### NavMesh
[官方文档](https://docs.unity3d.com/Manual/nav-MoveToDestination.html)
使用NavMesh来控制AI坦克的移动，使用SetDestination将Player设置为目标。
~~~C#
void FixedUpdate () {
		if(sceneController.getResult()==0) {
			target = sceneController.getPlayerPosition();
			if(HP.GetHP() <= 0.0f && recycleEvent != null) {
				recycleEvent(this.gameObject);
				sceneController.decreaseCountNPC();
			}
			else {
				NavMeshAgent agent = GetComponent<NavMeshAgent>();
				agent.SetDestination(target);
			}
		}
		else {
			NavMeshAgent agent = GetComponent<NavMeshAgent> ();
			agent.velocity = Vector3.zero;
			agent.ResetPath();
		}
	}
~~~

### 血条
学姐使用blood作为基类然后Player和NPC继承blood，bloodcontroller来控制血量，但是bloodcontroller完全没有使用，我将这部分代码重写，把上一次作业制作的预制体UGUI-H-Bar挂到玩家和电脑的坦克上，当子弹碰撞到时扣血即可。
~~~C#
using UnityEngine;
using System.Collections;
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