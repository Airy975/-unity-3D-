# 🎮 游戏项目说明文档

<!-- TOC -->
- [1. 游戏机制](#1-游戏机制)
  - [1.1. 核心玩法](#11-核心玩法)
  - [1.2. 战斗系统](#12-战斗系统)
  - [1.3. 需要注意的点](#13-需要注意的点)
- [2. 场景搭建](#2-场景搭建)
  - [2.1. 场景结构](#21-场景结构)
  - [2.2. 光照与氛围](#22-光照与氛围)
- [3. 游戏交互制作](#3-游戏交互制作)
  - [3.1. 输入系统](#31-输入系统)
  - [3.2. 交互反馈](#32-交互反馈)
- [4. 资源管理](#4-资源管理)
  - [4.1. 概述](#41-概述)
  - [4.2. 文件组织](#42-文件组织)
- [5. 开发环境](#5-开发环境)
- [6. 版本记录](#6-版本记录)
<!-- /TOC -->

---

## 1. 游戏机制

### 1.1. 核心玩法
游戏共有三关，主题分别为突袭推进、隐蔽渗透与协同作战。 第一关以突袭式推进为核心，采用固定路径卷轴推进结合弹幕射击玩法。玩家需操控坦克突破敌军防线与陷阱，护送步兵抵达指定区域。过程中需精准控制炮弹数量，合理设置补给与维修点，考验玩家的反应力与战略规划，避免盲目突进。 第二关以隐蔽与渗透为主，强调战术潜行。玩家需潜入敌占区，避开巡逻搜索，收集医疗物资和维修工具，用于维持部队与坦克的战斗状态。此关考验玩家的环境利用与隐蔽能力，强化生存与资源管理技巧。 第三关则转为协同与火力调度。玩家将驾驶修复后的坦克，与炮兵支援协同推进，对敌阵地展开正面攻势。需灵活标记目标、分配火力，在保持射击快感的同时体验战术指挥的成就感。三关节奏由快至稳，兼顾操作挑战与策略深度。 

---

## 2. 场景搭建
依据游戏的玩法以及剧情设定，同时为了呈现出真实的战争氛围，这款游戏总共精心设计了三个主要场景，分别是热带丛林战场，也就是第一关的场景，以及破败的越南村庄，这是第二关和第三关的场景。
游戏的第一关是固定路径卷轴推进结合弹幕射击玩法，这关的场景运用地图拼接的方式来呈现。本关地图的生成代码采用了Unity的Instantiate方法，借助周期性地图拼接实现动态生成场景，利用预制体Prefab进行设计，并且按照固定的Z轴间距依次排列，营造出不断向前推进的视觉感受，其基本逻辑用以下代码实现。

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlandLogic : MonoBehaviour
{
    public GameObject environmentPrefab;

    float interval = 5;
    // Start is called before the first frame update

    float times = 0;

    void Start()
    {
        InvokeRepeating("CreatePlane", 0, interval);

    }

    // Update is called once per frame
    void Update()
    {
        
    }

    void CreatePlane()
    {

        times = times + 1;

        GameObject node = Instantiate(environmentPrefab, Vector3.zero, Quaternion.identity);

        node.transform.position = new Vector3(0, 0, times * 60.35f);
    }

}


---

## 3. 游戏交互制作

### 3.1. 输入系统
- 支持键盘 / 鼠标 / 手柄 / 触控  
- 使用 Unity 新输入系统（Input System）  

### 3.2. 交互反馈
- 点击或碰撞时播放特效  
- UI 按钮动画与音效响应  
- 玩家操作提示与引导  

---

## 4. 资源管理

### 4.1. 概述
说明资源如何被组织与加载：
- 使用 Addressable / AssetBundle 动态加载  
- 音频、贴图、Prefab 的命名规范  

### 4.2. 文件组织
```bash
📁 Assets/
 ├── Scripts/           # 游戏脚本
 ├── Scenes/            # 场景文件
 ├── Prefabs/           # 预制体
 ├── Materials/         # 材质与贴图
 ├── UI/                # 界面资源
 └── Audio/             # 音效资源
