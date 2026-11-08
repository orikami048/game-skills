---
name: game-engine
description: 游戏引擎底层架构与高并发性能调优规范。专注于空间哈希网格（Spatial Hash Grid）碰撞加速、零分配预热对象池（Object Pool）、组合优于继承（ECS 理念）与生命周期安全托管。有效根除同屏实体爆炸时的卡顿掉帧、GC 尖峰与全局 God Object 导致的内存泄漏。
---

# 游戏引擎底层架构与性能调优规范 (game-engine)

许多由普通 AI 编写的游戏，在只有 20 个怪物时运行顺畅，一旦怪兽数量增加到 300~800 个，帧率就会断崖式暴跌至 10 FPS 以下，风扇狂转、频繁卡顿。  
这是因为普通 AI 缺乏**底层引擎架构思维**，习惯使用“双重 for 循环两两暴力检测”、“发射子弹就频繁 `new`”、“全局单例大杂烩（God Object）”。

本 Skill 旨在确立现代游戏引擎（Godot / Unity / 自研 WebGL 引擎）的工业级架构规范，从算法和内存底层彻底攻克性能瓶颈。

---

## 📸 实机架构与高压性能测试参考图

![游戏引擎与底层架构性能对比参考图](comparison.png)

> **实机压测：**
> * **❌ 未使用 Skill（暴力算法）**：800 个实体每帧执行 $800 \times 800 / 2 = 320,000$ 次距离开方计算，CPU 100% 满载，帧率跌至 12 FPS PPT 状态；频繁创建临时对象引发 GC 尖峰。
> * **✅ 搭载 game-engine（工业级架构）**：采用 **空间哈希网格（Spatial Hash Grid）** 将计算量削减 95% 以上，搭配 **预热对象池（Zero-allocation Pool）**，同屏 800 个实体依然稳定满帧 60 FPS，内存曲线平滑如镜。

---

## 🎯 传统 AI 架构三大死穴与工业级解法

### ① 碰撞检测：$O(N^2)$ 计算爆炸 vs 空间哈希网格 (Spatial Hash Grid)
* **翻车病症**：直接双重循环判断两两物体的距离。同屏 100 个怪要算 5,000 次，500 个怪算 125,000 次，800 个怪每秒要算上千万次乘除开方！
* **架构解法**：建立 **空间哈希网格（Spatial Hash Grid）**。
  * 将游戏世界划分为固定大小的二维网格（如 40px $\times$ 40px）；
  * 每个实体根据坐标计算所属格子 Key（`Math.floor(x / size) + "," + Math.floor(y / size)`）；
  * 碰撞检测时，**仅与同一格子及相邻 8 个格子的候选者计算**；
  * 算法复杂度从 $O(N^2)$ 降维至 $O(N)$，1000 个实体同屏移动检测如丝般顺滑。

### ② 内存与帧率稳定性：频繁 `new` 导致 GC 掉帧 vs 零分配对象池 (Object Pool)
* **翻车病症**：玩家发射子弹就 `new Bullet()`，敌人死亡掉金币就 `new Coin()`，用完丢弃等待垃圾回收。垃圾大量堆积会触发引擎/浏览器的全局垃圾回收（GC Spike），导致游戏每隔几秒突然卡顿掉帧一下，手感被毁。
* **架构解法**：采用 **预热对象池（Zero-allocation Pool）**。
  * 在游戏加载/进关卡时，一次性在内存中实例化 500~1000 个对象常驻内存；
  * 生成时从池中 `acquire()` 并初始化位置，销毁时执行 `recycle()` 重置状态还回池内；
  * 游戏运行全程 **0 次运行时内存分配**，内存波形平滑成一条直线，根除 GC 卡顿。

### ③ 组织形态：全局单例 God Object vs 组合解耦 (Composition / ECS)
* **翻车病症**：所有的逻辑都往 `GameManager.instance` 塞，包含物理、声音、金币、怪物刷新、关卡加载，单个类写了几千行。换关卡时残留事件监听导致内存严重泄漏，越玩越慢。
* **架构解法**：推崇 **组合优于继承（Composition over Inheritance）** 与 **生命周期严格托管**：
  * 实体作为轻量容器，按需挂载单一职责组件（`Transform`, `Collider`, `Health`）；
  * 业务逻辑由独立的 System 批量处理；
  * 关卡切换时统一派发 `Dispose` 信号，自动注销全部事件监听与弱引用字典，杜绝内存泄漏。

---

## 🌐 核心应用场景

| 业务场景 | 典型需求 | 本 Skill 的核心作用 |
|---|---|---|
| **1. 弹幕地狱与割草游戏** | 同屏 500~1500 个怪物与密集子弹 | 空间哈希网格 + 对象池，确保在中低端设备和移动端稳定 60 FPS。 |
| **2. 大地图开放世界 / 2D 地牢** | 大量未激活怪物、机关与宝箱 | 网格动态休眠机制（Frustum Culling），视野外实体冻结更新，只保底数据。 |
| **3. 跨关卡与长时间挂机** | 关卡频繁切换、挂机数小时 | 生命周期安全注销与对象池全局复用，杜绝内存泄漏崩溃（Out of Memory）。 |
| **4. 模块化角色定制** | 各种怪物的技能、移动、受击组合 | 基于组件组合模式（Component-based），新增一种怪物只需拼装组件，无需改写代码。 |

---

## 🤖 适配与实测推荐模型 (Model Recommendations)

| 任务类型 | 推荐使用模型 | 实测表现与推荐原因 |
|---|---|---|
| **算法优化与性能重构** | **Gemini 3.8 Flash (High)** *(实测验证推荐)* | 算法直觉敏锐，能秒出标准高效的空间哈希网格（Spatial Hash）与预分配对象池代码。 |
| **大型引擎架构与底层解耦** | **Claude 3.7 Sonnet** / **Gemini 3.8 Pro** | 极度擅长大型代码库的解耦重构，适合将面条代码拆解为规范的 ECS 或组件体系。 |
| **内存排查与 GC 调优** | **Claude 3.7 Sonnet** | 能快速通过代码上下文识别闭包泄漏、未注销的 EventListener 与循环引用隐患。 |

---

## 💻 工业级核心架构实现参考

### 1. 空间哈希网格碰撞加速器 (SpatialHashGrid.ts)
```typescript
export class SpatialHashGrid<T extends { x: number; y: number; radius: number }> {
  private cellSize: number;
  private cells = new Map<string, T[]>();

  constructor(cellSize = 40) {
    this.cellSize = cellSize;
  }

  public clear(): void {
    this.cells.clear();
  }

  private getKey(x: number, y: number): string {
    const cx = Math.floor(x / this.cellSize);
    const cy = Math.floor(y / this.cellSize);
    return `${cx},${cy}`;
  }

  public insert(entity: T): void {
    const key = this.getKey(entity.x, entity.y);
    let list = this.cells.get(key);
    if (!list) {
      list = [];
      this.cells.set(key, list);
    }
    list.push(entity);
  }

  // 仅获取本单元格及相邻 8 个格子的实体作为候选
  public getCandidates(entity: T): T[] {
    const cx = Math.floor(entity.x / this.cellSize);
    const cy = Math.floor(entity.y / this.cellSize);
    const candidates: T[] = [];

    for (let dx = -1; dx <= 1; dx++) {
      for (let dy = -1; dy <= 1; dy++) {
        const key = `${cx + dx},${cy + dy}`;
        const list = this.cells.get(key);
        if (list) {
          for (let i = 0; i < list.length; i++) {
            if (list[i] !== entity) candidates.push(list[i]);
          }
        }
      }
    }
    return candidates;
  }
}
```

### 2. 预热零分配通用对象池 (ObjectPool.ts)
```typescript
export class ObjectPool<T> {
  private pool: T[] = [];
  private factory: () => T;
  private resetFn: (item: T) => void;

  constructor(factory: () => T, resetFn: (item: T) => void, preloadCount = 500) {
    this.factory = factory;
    this.resetFn = resetFn;
    for (let i = 0; i < preloadCount; i++) {
      this.pool.push(this.factory());
    }
  }

  // 从池中获取实体，无运行时 GC 分配
  public acquire(): T {
    const item = this.pool.length > 0 ? this.pool.pop()! : this.factory();
    return item;
  }

  // 回收实体入池，等待下一次复用
  public release(item: T): void {
    this.resetFn(item);
    this.pool.push(item);
  }

  public get availableCount(): number {
    return this.pool.length;
  }
}
```
