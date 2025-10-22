# 🎮 游戏项目说明文档

<!-- TOC -->
- [1. 游戏机制](#1-游戏机制)
  - [1.1. 核心玩法](#11-核心玩法)
  - [1.2. 敌方机制](#12-敌方机制)
- [2. 场景搭建](#2-场景搭建)
  - [2.1. 场景结构](#21-场景结构)
  - [2.2. 光照与氛围](#22-光照与氛围)
- [3. 游戏交互制作](#3-游戏交互制作)
  - [3.1. 战斗交互](#31-战斗交互)
  - [3.2. 场景交互](#32-场景交互)
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

### 1.2. 敌方机制
第一关通过 `CreatorLogic` 脚本实现动态刷怪与物体生成系统。系统根据不同类型对象（敌人、障碍物、补给、Boss）设定独立的刷新时间，在玩家前方生成对应对象，实现持续战斗节奏与动态挑战体验。其基本逻辑用以下代码实现。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CreatorLogic : MonoBehaviour
{
    [Header("各类型刷新时间")]
    public float cannonInterval = 1.5f;       // 炮弹刷新间隔
    public float bulletInterval = 0.8f;        // 子弹刷新间隔
    public float healthItemInterval = 7f;      // 血包/弹药刷新间隔
    public float ObstaclesInterval = 3f;       // 障碍物刷新间隔
    public float BossInterval = 60f;            // Boss刷新时间

    [Header("生成对象")]
    public GameObject cannonPrefab;            // 炮弹Prefab
    public AudioClip cannonSound;            // 炮弹音效
    public GameObject bulletPrefab;            // 子弹Prefab
    public AudioClip bulletSound;            // 子弹音效
    public GameObject healthPrefab;            // 血量道具Prefab
    public GameObject ammoHealthPrefab;        // 弹药道具Prefab
    public GameObject Obstacles1;              // 障碍物1
    public GameObject Obstacles2;              // 障碍物2
    public GameObject Boss;                    // Boss
    private bool isBoss = false;                // 是否已经生成Boss
    private GameObject player;
    private GameObject shootTarget;

    void Start()
    {
        player = GameObject.FindWithTag("Player");
        shootTarget = GameObject.FindWithTag("shoot");


        // 三种不同类型分别开始自己的刷新
        InvokeRepeating(nameof(CreateCannon), 3f, cannonInterval);
        InvokeRepeating(nameof(CreateBullet), 1f, bulletInterval);
        InvokeRepeating(nameof(CreateHealthItem), 5f, healthItemInterval);
        InvokeRepeating(nameof(CreateObstacles), 0, ObstaclesInterval);
        Invoke(nameof(CreateBoss), BossInterval);
    }

    void CreateCannon()
    {
        if (player == null) return;

        if(isBoss == true) return;

        // 在玩家位置的前方随机生成一个炮弹
        Vector3 spawnPosition = player.transform.position + new Vector3(Random.Range(-11.5f, 12.5f), 1, 40);
        Quaternion spawnRotation = Quaternion.Euler(-90, 0, 0);

        Instantiate(cannonPrefab, spawnPosition, spawnRotation);

        AudioSource.PlayClipAtPoint(cannonSound, Camera.main.transform.position); // 在摄像机位置播放音效
    }

    void CreateBullet()
    {
        if (player == null) return;

        float randomX = Random.Range(0, 2) == 0 ? Random.Range(-50f, -20f) : Random.Range(20f, 50f);
        float randomZ = Random.Range(10f, 30f);
        Vector3 spawnPosition = player.transform.position + new Vector3(randomX, 1, randomZ);

        GameObject bullet = Instantiate(bulletPrefab, spawnPosition, Quaternion.identity);
        if (shootTarget != null)
        {
            bullet.transform.LookAt(shootTarget.transform);
        }

        AudioSource.PlayClipAtPoint(bulletSound, Camera.main.transform.position);
    }

    void CreateHealthItem()
    {
        if (player == null) return;

        Vector3 spawnPosition = player.transform.position + new Vector3(Random.Range(-11.5f, 12.5f), 1.4f, 40);
        int sel = Random.Range(0, 2);
        GameObject selectedPrefab = (sel == 0) ? healthPrefab : ammoHealthPrefab;

        Instantiate(selectedPrefab, spawnPosition, Quaternion.identity);
    }

    void CreateObstacles()
    {
        if (player == null) return;

        if(isBoss == true) return;

        int sel = Random.Range(0, 2);
        GameObject selectedPrefab = (sel == 0) ? Obstacles1 : Obstacles2;

        float yPos = (sel == 0) ? 3.86f : -0.82f; // 根据选择的障碍物设置不同的y值

        float randomZ = Random.Range(35, 55);

        Vector3 spawnPosition = player.transform.position + new Vector3(Random.Range(-9, 8), yPos, randomZ);

        Instantiate(selectedPrefab, spawnPosition, Quaternion.identity);
    }

    void CreateBoss()
    {
        if (player == null) return;

        Vector3 spawnPosition = player.transform.position + new Vector3(0, 0, 27);
        Quaternion spawnRotation = Quaternion.Euler(0, -180, 0);

        Instantiate(Boss, spawnPosition, spawnRotation);

        isBoss = true;
    }
}
```

`BossLogic` 是 Boss 战的核心控制脚本，负责实现以下功能：

- 前进逻辑：Boss 持续沿 Z 轴与玩家保持距离。  
- 炮塔追踪：炮塔自动旋转，始终朝向玩家。  
- 子弹发射：定时生成 Boss 子弹（间隔 1.5 秒）。  
- 受击检测：当 Boss 被“炮弹”击中时执行受伤逻辑。  

通过该机制，Boss 在场景中具备持续威胁性和动态战斗表现。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class BossLogic : MonoBehaviour
{
    public Transform TankTurret;      // Boss 炮塔部分
    public Transform player;          // 玩家目标
    public GameObject BossBullet;     // Boss 子弹预制体
    public Transform firePoint;       // 子弹发射点
    public float FrontSpeed = 5;      // 前进速度
    public float rotationSpeed = 5f;  // 炮塔旋转速度

    void Start()
    {
        // 每 1.5 秒发射一枚子弹
        InvokeRepeating(nameof(BulletCreat), 0, 1.5f);

        // 获取玩家引用
        player = GameObject.FindWithTag("Player").transform; 
    }

    void Update()
    {
        // Boss 持续前进
        transform.position = new Vector3(
            transform.position.x,
            transform.position.y,
            transform.position.z + FrontSpeed * Time.deltaTime
        );

        // 炮塔自动朝向玩家
        if (player != null)
        {
            Vector3 direction = player.position - TankTurret.position;
            direction.y = 0f;

            if (direction != Vector3.zero)
            {
                Quaternion targetRotation = Quaternion.LookRotation(direction);
                TankTurret.rotation = Quaternion.Slerp(
                    TankTurret.rotation,
                    targetRotation,
                    rotationSpeed * Time.deltaTime
                );
            }
        }
    }

    /// <summary>
    /// 创建子弹（Boss 攻击逻辑）
    /// </summary>
    public void BulletCreat()
    {
        // 在发射点生成子弹
        GameObject node = Instantiate(BossBullet, firePoint.position, firePoint.rotation);
        node.transform.position = firePoint.position;
    }

    private void OnTriggerEnter(Collider other)
    {
        PlayerHealthLogic playHealth = this.gameObject.GetComponent<PlayerHealthLogic>();

        // 当被“炮弹”击中时受伤
        if (other.name.StartsWith("炮弹"))
        {
            Destroy(other.gameObject);
            playHealth.TakeDamage(5);
            return;
        }
    }
}
```

---

## 2. 场景搭建
依据游戏的玩法以及剧情设定，同时为了呈现出真实的战争氛围，这款游戏总共精心设计了三个主要场景，分别是热带丛林战场，也就是第一关的场景，以及破败的越南村庄，这是第二关和第三关的场景。
游戏的第一关是固定路径卷轴推进结合弹幕射击玩法，这关的场景运用地图拼接的方式来呈现。本关地图的生成代码采用了Unity的Instantiate方法，借助周期性地图拼接实现动态生成场景，利用预制体Prefab进行设计，并且按照固定的Z轴间距依次排列，营造出不断向前推进的视觉感受，其基本逻辑用以下代码实现。

```csharp
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
```

第二关与第三关所运用的是同一个场景，这个场景借助unity的Terrain地形系统来进行搭建，先是构建出道路、河流山川这类自然景观，接着还添加了树木、花草等细节。运用Terrain当中的Raise/Lower Terrain以及Paint Height等工具去塑造村庄的地形，道路、河流附近区域的高低差，以此提高场景的真实感，借助Paint Texture工具，绘制出草地、路面以及沙石等多种不同的纹理，模拟出真实的越南村庄模样。运用Paint Tree/Detail功能，添加杂草、花朵、树木等植被，凸显出热带丛林的复杂特性

---

## 3. 游戏交互制作

### 3.1. 战斗交互
- 支持键盘 / 鼠标 / 手柄 / 触控  
- 使用 Unity 新输入系统（Input System）  

### 3.2. 场景交互
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
