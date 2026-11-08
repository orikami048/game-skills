# 游戏制作技能库 (Game Skills) 🎮

> 面向独立游戏与小游戏开发者的 **AI 游戏开发专项 Skill 体系**。  
> 告别普通 AI 写的“塑料玩具代码”与“假像素缝合怪”，将工业级**游戏核心循环、正统 16-Bit 界面规范、真实物理打击感（Game Feel）**封装为可直接挂载的专家级 Agent Skills。

---

## 📸 实机对比与效果展示 (Visual Comparisons)

本项目所有 Skill 均经过严格的场景化实机测试，以下为挂载对应 Skill 与普通 AI 生成效果的残酷对比：

### 1. `game-ui-design`：16-Bit 像素游戏 UI 实机动图演示

![16-Bit UI 实机对比动图](assets/ui_comparison.gif)

* **❌ 左侧（未使用 Skill - 普通 AI 常见作派）**：现代矢量平滑圆角、平滑渐变血条、CSS3 模糊投影、0.3s 缓动悬浮动画，呈现浓浓的“伪像素/现代网页缝合怪”违和感。
* **✅ 右侧（搭载 `game-ui-design` 工业级规范）**：
  * **SFC 经典双层金属边框**：纯代码多重阴影嵌套，严格 2px/4px 整数倍点阵咬合；
  * **阶梯像素血条**：3 级定值色阶表达体积，拒绝任何平滑渐变；
  * **街机微动手感**：0ms 渐变延迟，按压纯物理**下沉 4px**，高光与阴影瞬态反转；
  * **复古滤镜支持**：内置可选 CRT 显像管扫描线（Scanlines）光栅。
* 🎮 **本地交互试玩原型**：双击运行 [`prototypes/ui_showcase.html`](prototypes/ui_showcase.html)

---

### 2. `game-feel`：动作游戏打击感与微反馈实机动态对比（全循环动图）

![Game Feel 命中受击实机高燃动图对比](assets/feel_comparison.gif)

* **❌ 前半段（未使用 Skill）**：只有简单的伤害数字匀速直飘与粗暴扣血，无阻力、无顿挫，轻飘飘软绵绵如切薄纸。
* **✅ 后半段（搭载 `game-feel` 工业级神经生理欺骗系统）**：
  * **顿帧定格（Hitstop: 75ms）**：命中瞬间全局时空冻结 4~5 帧，赋予兵刃切入骨肉的硬核物理阻力；
  * **受击纯白闪烁（White Flash）**：首帧纯白高亮 2 帧，极强视网膜刺激；
  * **创伤衰减震屏（Trauma² Shake）**：基于 Vlambeer 公式，微击微颤、重击剧烈，带角位移微偏转；
  * **弹性挤压拉伸（Squash & Stretch）**：受击瞬间横向拉宽 **1.45 倍**、纵向压扁，呈现肉质弹性受力；
  * **切向爆裂火花**：沿挥砍法线锥形喷涌 28 颗高初速重力火花；
  * **声学协同**：纯数学合成低频钝击沉闷重音（Impact Thud）与破裂噪音。
* 🎮 **本地交互试玩原型**：双击运行 [`prototypes/feel_showcase.html`](prototypes/feel_showcase.html)（支持按空格键挥刀、自由开关顿帧/震屏/白闪）

---

## 🤖 适配与实测推荐模型矩阵 (Model Matrix)

为确保 Skills 发挥最佳效果，针对不同开发环节推荐选用最匹配的模型：

| 开发任务环节 | 推荐使用模型 | 实测表现与推荐原因 |
|---|---|---|
| **代码实现与手感逻辑** | **Gemini 3.8 Flash (High)**<br>*(实操验证推荐)* | 响应迅速，逻辑严密。能准确理解 60FPS 游戏循环中的顿帧挂起、阻尼摩擦力数学公式与 0ms 像素位移。 |
| **高复杂度底层系统与网络** | **Claude 3.7 Sonnet** / **Gemini 3.8 Pro** | 适合生成复杂的 Modifier 属性管线、Godot/Unity HLSL 像素着色器以及多人帧同步/预测回滚逻辑。 |
| **像素资产出图 (Sprite/UI)** | **Imagen 3 (Fast / HQ)** / **Gemini 3.8 Flash ImageGen** | 对 `#FF00FF` 品红底与“无外发光、无抗锯齿”负面提示词执行力强，网格漂移率极低。 |
| **风格锚点与概念原画** | **Midjourney v6 (`--style raw`)** | 适合立项初期确立复古调色盘与全局艺术基调。 |

---

## 📂 项目结构

```text
game-skills/
├── README.md                      # 项目简介（实机对比、模型推荐与试玩导航）
├── assets/                        # 高清对比截图
│   ├── ui_comparison.png          # 16-Bit UI 左右分屏实机对比
│   └── feel_comparison.png        # Game Feel 高能打击感命中实机图
├── prototypes/                    # 独立交互试玩原型 (可直接在浏览器中打开试玩)
│   ├── ui_showcase.html           # 16-Bit UI 交互原型 (SFC 边框/4px下沉/扫描线)
│   └── feel_showcase.html         # 60FPS 战斗打击面试验台 (带音效与参数开关)
└── skills/                        # 核心技能正本
    ├── game-ui-design/            # 16-Bit 游戏 UI 规范与九宫格边框
    │   ├── SKILL.md
    │   ├── comparison.png
    │   └── showcase.html
    ├── game-feel/                 # 游戏手感、顿帧、震屏与受击反馈算法
    │   ├── SKILL.md
    │   ├── comparison.png
    │   └── showcase.html
    └── higgsfield-game-generation/# 核心玩法生成、状态机闭环与 Modifier 属性管线
        └── SKILL.md
```

---

## 🚀 安装与挂载

### 1. Google Antigravity (Gemini)
复制技能目录到全局配置即可自动热加载：

```powershell
Copy-Item -Recurse -Force skills\* $env:USERPROFILE\.gemini\config\skills\
```

### 2. Claude Code
复制技能目录到用户级 skills 目录：

```powershell
Copy-Item -Recurse -Force skills\* $env:USERPROFILE\.claude\skills\
```

---

## 📜 许可证
MIT License. 欢迎独立游戏开发者与小游戏创作者自由使用与扩展！
