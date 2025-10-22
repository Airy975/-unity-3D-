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

## 1.2. 敌方机制

---

### 第一关：动态生成与 Boss 战

#### CreatorLogic：动态刷怪与物体生成系统
第一关通过 `CreatorLogic` 脚本实现动态刷怪与物体生成系统。  
系统根据不同类型对象（敌人、障碍物、补给、Boss）设定独立的刷新时间，在玩家前方生成对应对象，实现持续战斗节奏与动态挑战体验。  

核心功能：
炮弹、子弹、道具、障碍物、Boss 分别按独立时间间隔生成  
Boss 在关卡后期出现并触发战斗阶段  

#### 核心逻辑代码
```csharp
void Start()
{
    player = GameObject.FindWithTag("Player");
    shootTarget = GameObject.FindWithTag("shoot");

    // 各类型对象按独立时间间隔生成
    InvokeRepeating(nameof(CreateCannon), 3f, cannonInterval);
    InvokeRepeating(nameof(CreateBullet), 1f, bulletInterval);
    InvokeRepeating(nameof(CreateHealthItem), 5f, healthItemInterval);
    InvokeRepeating(nameof(CreateObstacles), 0, ObstaclesInterval);

    // Boss 在指定时间后出现
    Invoke(nameof(CreateBoss), BossInterval);
}
```

#### 生成逻辑示例
炮弹生成（定点前方 + 音效）
```csharp
void CreateCannon()
{
    if (player == null || isBoss) return;

    Vector3 pos = player.transform.position + new Vector3(Random.Range(-11.5f, 12.5f), 1, 40);
    Instantiate(cannonPrefab, pos, Quaternion.Euler(-90, 0, 0));
    AudioSource.PlayClipAtPoint(cannonSound, Camera.main.transform.position);
}
```

子弹生成（两侧随机 + 瞄准玩家）
```csharp
void CreateBullet()
{
    if (player == null) return;

    float randomX = Random.Range(0, 2) == 0 ? Random.Range(-50f, -20f) : Random.Range(20f, 50f);
    Vector3 pos = player.transform.position + new Vector3(randomX, 1, Random.Range(10f, 30f));
    GameObject bullet = Instantiate(bulletPrefab, pos, Quaternion.identity);
    bullet.transform.LookAt(shootTarget.transform);
}
```

道具与障碍物生成
```csharp
void CreateHealthItem()
{
    Vector3 pos = player.transform.position + new Vector3(Random.Range(-11.5f, 12.5f), 1.4f, 40);
    GameObject prefab = Random.Range(0, 2) == 0 ? healthPrefab : ammoHealthPrefab;
    Instantiate(prefab, pos, Quaternion.identity);
}

void CreateObstacles()
{
    if (isBoss) return;

    GameObject prefab = Random.Range(0, 2) == 0 ? Obstacles1 : Obstacles2;
    float yPos = prefab == Obstacles1 ? 3.86f : -0.82f;
    Vector3 pos = player.transform.position + new Vector3(Random.Range(-9, 8), yPos, Random.Range(35, 55));
    Instantiate(prefab, pos, Quaternion.identity);
}
```

#### Boss登场阶段
```csharp
void CreateBoss()
{
    Vector3 pos = player.transform.position + new Vector3(0, 0, 27);
    Instantiate(Boss, pos, Quaternion.Euler(0, -180, 0));
    isBoss = true;
}
```

####  BossLogic：Boss 行为机制

BossLogic是第一关 Boss 战的核心控制脚本，负责实现以下功能：
前进逻辑：Boss 持续沿 Z 轴与玩家保持距离
炮塔追踪：炮塔自动旋转，始终朝向玩家
子弹发射：定时生成 Boss 子弹（间隔 1.5 秒）
受击检测：当 Boss 被“炮弹”击中时执行受伤逻辑

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


### 🕹 第二关：敌人听觉感知机制
在第二关中，敌人具备听觉侦测能力。  
当玩家在一定范围内制造声响（例如射击、奔跑或触发机关）时，敌人会通过监听事件 `OnHearSound()` 来做出反应。  
一旦敌人听到声音，游戏会立即判定玩家暴露并触发失败UI，其基本逻辑用以下代码实现。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class EnemyHearing : MonoBehaviour
{
    [Header("游戏失败UI")]
    public GameObject gameOverUI;

    [Header("是否只触发一次")]
    public bool onlyTriggerOnce = true;

    private bool hasTriggered = false;
    // Start is called before the first frame update
    public void OnHearSound(object[] data)
    {
        if (hasTriggered && onlyTriggerOnce) return;

        Vector3 soundPosition = (Vector3)data[0];
        float volume = (float)data[1];

        Debug.Log("听到了声音！");

        TriggerGameOver();
        hasTriggered = true;
    }

    void TriggerGameOver()
    {
        if (gameOverUI != null)
        {
            gameOverUI.SetActive(true);
        }

        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
        Time.timeScale = 0f; // 暂停游戏

        Debug.Log("游戏失败！");
    }

}
```

### 第三关：多类型敌人机制
第三关有两种不同的敌人类型，以增强战斗节奏与战术多样性。

#### EnemySoldierLogic：士兵逻辑
其主要功能为：
- 自动侦测玩家：在检测范围内持续攻击。  
- 自动旋转朝向玩家并定时射击。  
- 可被击杀并触发死亡动画。
其基本逻辑用以下代码实现。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemySoldierLogic : MonoBehaviour
{
    public Transform player;
    public GameObject bulletPrefab;
    public Transform firePoint;
    public float fireInterval = 3f;
    public float detectionRange = 15f;

    private float fireTimer = 0f;
    private Animator animator;
    private bool isDead = false;
    // Start is called before the first frame update
    void Start()
    {
        animator = GetComponent<Animator>();
    }

    // Update is called once per frame
    void Update()
    {
        if (isDead || player == null) return;

        float distance = Vector3.Distance(transform.position, player.position);

        if (distance <= detectionRange)
        {
            // 1. 始终面向主角
            Vector3 lookDir = player.position - transform.position;
            lookDir.y = 0;  // 保持水平旋转
            transform.rotation = Quaternion.Slerp(transform.rotation, Quaternion.LookRotation(lookDir), Time.deltaTime * 5f);

            // 2. 每隔 fireInterval 秒发射一发子弹
            fireTimer += Time.deltaTime;
            if (fireTimer >= fireInterval)
            {
                FireBullet();
                fireTimer = 0f;
            }
        }
    }

    void FireBullet()
    {
        if (bulletPrefab != null && firePoint != null)
        {
            GameObject bullet = Instantiate(bulletPrefab, firePoint.position, firePoint.rotation);
        }
    }

    public void TakeDamage()
    {
        Die();
    }

    void Die()
    {
        isDead = true;
        animator.SetBool("isDead", true);

        Invoke("StartFall", 1.5f);

        // 可选：停止移动、停止射击，销毁子弹发射等
        Destroy(this.gameObject, 3f); // 3 秒后销毁敌人
    }

    void StartFall()
    {
        StartCoroutine(SlowFallToGround());
    }

    IEnumerator SlowFallToGround()
    {
        float duration = 1f;
        float elapsed = 0f;
        Vector3 startPos = transform.position;
        Vector3 endPos = startPos;
        endPos.y -= 0.8f;  // 你可以根据情况调大，例如 1.5f 或 2f

        while (elapsed < duration)
        {
            transform.position = Vector3.Lerp(startPos, endPos, elapsed / duration);
            elapsed += Time.deltaTime;
            yield return null;
        }

        transform.position = endPos;
    }


}
```
#### FortLogic：固定堡垒逻辑
其主要功能为：
- 面向玩家的血条UI。  
- 根据玩家方向选择炮口进行攻击。
- 使用多炮口射击（前、后、左、右）。
- 拥有独立生命值与受击反馈。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class FortLogic : MonoBehaviour
{
    public Transform player;

    [Header("UI系统")]
    private float StartingHealth = 100;
    private float CurrentHeatHealth;
    private bool isDead = false;
    public Transform healthUI;
    public Slider slider;
    public Image fill;
    public Color FullHealthColor = Color.green;
    public Color ZeroHealthColor = Color.red;

    [Header("发射系统")]
    public Transform[] frontFirePoints;
    public Transform[] rightFirePoints;
    public Transform[] backFirePoints;
    public Transform[] leftFirePoints;
    public GameObject bulletPrefab;
    public GameObject cannonPrefab;
    public float fireInterval = 3f;

    private float fireTimer = 0f;

    private void OnEnable()
    {
        CurrentHeatHealth = StartingHealth;
        isDead = false;
        slider.value = CurrentHeatHealth;
    }

    public void ForkDamage(float amount)
    {
        CurrentHeatHealth -= amount;
        slider.value = CurrentHeatHealth;
        fill.color = Color.Lerp(ZeroHealthColor, FullHealthColor, slider.normalizedValue);

        if (CurrentHeatHealth <= 0 && !isDead)
        {
            isDead = true;
            gameObject.SetActive(false);
        }
    }

    void Update()
    {
        // 让血条UI朝向玩家
        Vector3 direction = player.position - healthUI.position;
        direction.y = 0f;
        healthUI.rotation = Quaternion.LookRotation(direction);

        float distanceToPlayer = Vector3.Distance(transform.position, player.position);
        if (distanceToPlayer > 20f) return;

        fireTimer += Time.deltaTime;
        if (fireTimer >= fireInterval)
        {
            FireInPlayerDirection();
            fireTimer = 0f;
        }
    }

    private void FireInPlayerDirection()
    {
        Vector3 toPlayer = player.position - transform.position;
        toPlayer.y = 0f;
        toPlayer.Normalize();

        float angle = Vector3.SignedAngle(transform.forward, toPlayer, Vector3.up);

        if (angle >= -45f && angle < 45f)
            FireFromRandom(frontFirePoints);
        else if (angle >= 45f && angle < 135f)
            FireFromRandom(rightFirePoints);
        else if (angle >= -135f && angle < -45f)
            FireFromRandom(leftFirePoints);
        else
            FireFromRandom(backFirePoints);
    }

    void FireFromRandom(Transform[] firePoints)
    {
        if (firePoints == null || firePoints.Length == 0 || player == null) return;

        Transform selectedPoint = firePoints[Random.Range(0, firePoints.Length)];
        Vector3 directionToPlayer = (player.position - selectedPoint.position).normalized;
        Quaternion lookRotation = Quaternion.LookRotation(directionToPlayer);

        GameObject prefabToFire = (Random.value > 0.5f) ? bulletPrefab : cannonPrefab;
        if (prefabToFire == null) return;

        Instantiate(prefabToFire, selectedPoint.position, lookRotation);
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.name.StartsWith("炮弹"))
        {
            Destroy(other.gameObject);
            ForkDamage(10);
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
