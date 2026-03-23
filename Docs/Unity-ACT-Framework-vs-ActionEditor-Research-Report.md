# 研究报告：Unity-ACT-framework 与 ActionEditor 深度对比分析

## 1. 概述

本报告对以下两个 Unity 动作/时间轴相关开源仓库进行深入对比分析：

| 项目 | Unity-ACT-framework | ActionEditor (NBC.ActionEditor) |
|------|---------------------|--------------------------------|
| 仓库地址 | [Chenxi-Zhang/Unity-ACT-framework](https://github.com/Chenxi-Zhang/Unity-ACT-framework) | [NoBugCn/ActionEditor](https://github.com/NoBugCn/ActionEditor) |
| 作者 | Chenxi Zhang | NoBug (BobSong) |
| 许可证 | MIT | 未明确标注 |
| 最新更新 | 2026年1月 | 2025年7月 |
| 项目定位 | 完整的 ACT 游戏框架 | 通用行为时间轴编辑器工具 |

---

## 2. 项目定位与目标

### 2.1 Unity-ACT-framework

**定位**：一个专为动作游戏（ACT）设计的 **完整游戏框架**。

核心理念是将角色的所有动作（攻击、移动、防御等）通过 Unity Timeline 来编排，替代传统动画状态机方案。框架包含战斗、移动、相机、输入、AI 等完整的游戏子系统，是一个可以直接用于开发动作游戏的框架级项目。

**目标用户**：想要快速构建动作游戏原型或学习 ACT 框架设计的 Unity 开发者。

### 2.2 ActionEditor

**定位**：一个通用的 **行为时间轴编辑器工具**。

核心理念是提供一个纯编辑器层面的时间轴可视化编辑工具，不包含任何具体业务逻辑。用户通过继承和扩展来定义自己的 Asset、Track、Clip 类型，从而实现技能编辑器、Buff 编辑器、场景动画编辑器等各种可视化编辑需求。

**目标用户**：需要为自己的项目构建时间轴编辑工具的 Unity 开发者/技术策划。

---

## 3. 架构设计对比

### 3.1 Unity-ACT-framework 架构

```
┌─────────────────────────────────────────────────┐
│                     Scene                        │
│  ┌──────────── Actor (核心控制器) ─────────────┐  │
│  │  ┌─────────────────┐  ┌──────────────────┐  │  │
│  │  │  ActorLogicInput │  │ ActionPlayable   │  │  │
│  │  │  (输入系统)       │  │ Director (时间轴) │  │  │
│  │  └────────┬────────┘  └────────┬─────────┘  │  │
│  │           │                    │             │  │
│  │  ┌────────▼────────┐  ┌───────▼──────────┐  │  │
│  │  │  ActorMovement   │  │ AnimationSimple  │  │  │
│  │  │  (移动系统)       │  │ Blender (动画)   │  │  │
│  │  └─────────────────┘  └──────────────────┘  │  │
│  │                                              │  │
│  │  ┌─────────────────┐  ┌──────────────────┐  │  │
│  │  │ AttackCollider   │  │ ActorCameraStatus│  │  │
│  │  │ Manager (碰撞)   │  │ (相机状态)       │  │  │
│  │  └─────────────────┘  └──────────────────┘  │  │
│  │                                              │  │
│  │  ┌─────────────────┐  ┌──────────────────┐  │  │
│  │  │  ActorBeHit      │  │  ActorAI         │  │  │
│  │  │  (受击系统)       │  │  (AI系统)        │  │  │
│  │  └─────────────────┘  └──────────────────┘  │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌──────────────────┐  ┌────────────────────────┐  │
│  │PlayerInputControl│  │ Camera (Cinemachine)   │  │
│  │ler (玩家输入)     │  │ (相机系统)             │  │
│  └──────────────────┘  └────────────────────────┘  │
└────────────────────────────────────────────────────┘

数据层：ScriptableObject 配置
  ├── ActionTimelineAsset (Timeline 资源封装)
  ├── AttackData (攻击配置)
  ├── DefenceData (防御配置)
  ├── HitCounterConfig (反击配置)
  ├── ShockConfig (震荡配置)
  └── StrafeMoveAnimation (移位动画映射)
```

**关键设计特点**：
- **组件化设计**：Actor 作为中心控制器，管理多个子系统组件
- **基于 Unity Timeline 的 Playable 系统**：使用 Unity 内置 PlayableDirector 控制 Timeline 播放
- **手动帧控制**：自定义 `ActionPlayableDirector`，使用 `DirectorUpdateMode.Manual` 实现精确的帧级执行控制
- **事件驱动**：系统间通过事件（如 `onActionDone`）通信
- **输入优先级队列**：`InputType` 枚举值同时代表优先级，高优先级动作自动打断低优先级

### 3.2 ActionEditor 架构

```
┌─────────────────────────────────────────────────────┐
│              Editor (编辑器层)                        │
│  ┌──────────────────────────────────────────────┐   │
│  │  App (静态管理器)                              │   │
│  │  ├── AssetData (当前编辑的资产)                 │   │
│  │  ├── Select (选择管理)                         │   │
│  │  ├── Copy/Cut (剪贴板)                         │   │
│  │  ├── Play/Stop/Pause (播放控制)                │   │
│  │  └── AutoSave (自动保存)                       │   │
│  └──────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────┐   │
│  │  ActionEditorWindow (主编辑窗口)                │   │
│  │  ├── Views (时间轴视图)                        │   │
│  │  ├── GUIS/Customized (自定义界面)              │   │
│  │  └── Preview (预览)                            │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│              Runtime (运行时层)                       │
│  ┌──────────────── 数据模型 ────────────────────┐   │
│  │                                               │   │
│  │  Asset (资产) ──┬── Group (分组)               │   │
│  │                 └── Group ──┬── Track (轨道)   │   │
│  │                             └── Track ──┬── Clip│  │
│  │                                         └── Clip│  │
│  └───────────────────────────────────────────────┘   │
│  ┌──────────────── 接口体系 ────────────────────┐   │
│  │  IDirector ← IData                           │   │
│  │  IDirectable ← IData                         │   │
│  │  IClip ← IDirectable                         │   │
│  │  ISubClipContainable                          │   │
│  └───────────────────────────────────────────────┘   │
│  ┌──────────────── 序列化 ──────────────────────┐   │
│  │  JSON (FullSerializer) ↔ TextAsset            │   │
│  └───────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

**关键设计特点**：
- **纯工具层设计**：不包含任何游戏业务逻辑，只提供编辑器框架
- **四层数据模型**：Asset → Group → Track → Clip 层次结构
- **接口驱动**：通过 `IDirectable`、`IDirector` 等接口定义行为契约
- **JSON 序列化**：使用 FullSerializer 将数据存储为 JSON TextAsset
- **特性（Attribute）驱动扩展**：通过 `[Name]`、`[Attachable]`、`[Color]`、`[Category]` 等特性配置扩展行为
- **Unity Package 形式**：以 UPM 包方式分发，方便集成

---

## 4. 核心功能对比

### 4.1 时间轴系统

| 维度 | Unity-ACT-framework | ActionEditor |
|------|---------------------|--------------|
| 时间轴引擎 | **Unity 原生 Timeline** (PlayableDirector) | **自研时间轴系统** (JSON数据模型) |
| 数据存储 | Unity TimelineAsset (.playable) | JSON TextAsset |
| 编辑界面 | Unity 原生 Timeline 编辑器 | **自研编辑器窗口** (EditorWindow) |
| 扩展方式 | 自定义 PlayableAsset/PlayableBehaviour | 继承 Asset/Group/Track/Clip |
| 运行时驱动 | Unity PlayableGraph | 自研 AssetPlayer 采样系统 |
| 多轨道支持 | 原生多轨道 | 自定义 Group-Track-Clip 层级 |
| 混合/过渡 | Unity 原生 BlendIn/BlendOut | 自研 CrossBlend 系统 |

**分析**：

Unity-ACT-framework 直接使用 Unity 原生 Timeline，优势是成熟稳定、编辑器功能完善、与 Unity 生态深度集成；缺点是扩展灵活性受限于 Unity Timeline API。

ActionEditor 完全自研时间轴系统，优势是极高的自定义灵活性、不依赖 Unity Timeline；缺点是需要自行维护编辑器和运行时，功能成熟度不及 Unity Timeline。

### 4.2 游戏系统覆盖

| 游戏系统 | Unity-ACT-framework | ActionEditor |
|----------|---------------------|--------------|
| 战斗系统 | ✅ 完整（攻击、防御、反击） | ❌ 不包含 |
| 移动系统 | ✅ 完整（RootMotion、Strafe） | ❌ 不包含 |
| 相机系统 | ✅ 完整（锁定、跟随、Cinemachine）| ❌ 不包含 |
| 输入系统 | ✅ 完整（缓冲、优先级队列） | ❌ 不包含 |
| AI 系统 | ✅ 基础（NodeCanvas集成） | ❌ 不包含 |
| 碰撞检测 | ✅ 完整（攻击判定框） | ❌ 不包含 |
| 技能编辑器 | ❌ 使用原生 Timeline 编辑器 | ✅ 核心功能 |
| 通用行为编辑 | ❌ 专用于ACT | ✅ 核心功能 |
| 预览系统 | ❌ Unity Timeline 内置预览 | ✅ 自研预览系统 |

### 4.3 数据驱动与配置

| 维度 | Unity-ACT-framework | ActionEditor |
|------|---------------------|--------------|
| 配置方式 | ScriptableObject | JSON (FullSerializer) |
| 编辑器集成 | Unity Inspector | 自研 Inspector 面板 |
| 序列化格式 | Unity 原生序列化 | JSON 文本 |
| 版本控制友好度 | 一般（YAML序列化） | 好（纯JSON文本） |
| 热重载 | 原生支持 | 需要重新加载 |

---

## 5. 代码设计模式对比

### 5.1 Unity-ACT-framework 的设计模式

1. **组件模式 (Component Pattern)**
   ```
   Actor 持有多个子系统引用：
   - ActorLogicInput (输入)
   - ActorMovement (移动)
   - ActionPlayableDirector (时间轴)
   - AnimationSimpleBlender (动画混合)
   - ActorAttackColliderManager (攻击碰撞)
   - ActorCameraStatus (相机状态)
   ```

2. **手动更新模式**
   ```csharp
   // Actor.Update() 中手动调用各子系统的更新
   void Update() {
       logicInput.DoUpdate(deltaTime);
       actionPlayableDirector.DoUpdate(deltaTime);
       movement.DoUpdate(deltaTime);
       animationSimpleBlender.DoUpdate(deltaTime);
   }
   ```

3. **优先级输入系统**
   ```csharp
   // InputType 枚举值即优先级，大值优先
   // LateUpdate 中取最高优先级输入执行
   ```

4. **ScriptableObject 单例配置**
   ```
   HitCounterConfig, ShockConfig 等使用 SingletonHolder 管理
   ```

### 5.2 ActionEditor 的设计模式

1. **模板方法模式 (Template Method)**
   ```csharp
   // 通过抽象基类定义流程，子类重写具体行为
   public abstract class Clip : IClip {
       public virtual float Length { get; set; }
       public virtual string Info { get; }
       public virtual bool IsValid => false;
   }
   ```

2. **访问者/验证模式 (Visitor/Validation)**
   ```csharp
   // Asset.Validate() 递归遍历所有层级进行验证
   public void Validate() {
       foreach (group in groups)
           group.Validate(this, null);
           foreach (track in group.Children)
               track.Validate(this, group);
               foreach (clip in track.Children)
                   clip.Validate(this, track);
   }
   ```

3. **特性驱动扩展 (Attribute-Driven)**
   ```csharp
   [Name("普通粒子片段")]
   [Color(0.0f, 1f, 1f)]
   [Attachable(typeof(EffectTrack))]
   public class TestClip : ActionClip { }
   ```

4. **静态应用管理器 (Static App Manager)**
   ```csharp
   // App 静态类管理全局状态
   App.AssetData  // 当前编辑资产
   App.Select()   // 选择管理
   App.Play()     // 播放控制
   App.SaveAsset() // 保存
   ```

---

## 6. 扩展性对比

### 6.1 Unity-ACT-framework

**扩展方式**：
- 通过自定义 Timeline Clip（PlayableAsset + PlayableBehaviour）扩展时间轴行为
- 通过新增 MonoBehaviour 组件扩展 Actor 子系统
- 通过新增 ScriptableObject 添加配置数据类型

**扩展难度**：中等。需要理解 Unity Timeline/Playable 系统和框架的事件通信机制。

**扩展灵活性**：受限于 Unity Timeline 的设计限制。例如 Timeline 的 Track 类型需要与 Unity 编辑器集成，自定义编辑器体验较为复杂。

### 6.2 ActionEditor

**扩展方式**：
- 继承 `Asset` 创建自定义资产类型
- 继承 `Group` 创建自定义分组类型
- 继承 `Track` 创建自定义轨道类型
- 继承 `Clip`/`ClipSignal`/`ClipCrossBlend` 创建自定义片段类型
- 通过特性（Attribute）配置显示名称、颜色、关联关系

**扩展难度**：低。文档完善，示例代码清晰，只需少量代码即可完成扩展。

**扩展灵活性**：极高。从资产到片段的每一层都可以自定义，编辑器界面也提供大量重写接口。

```csharp
// 示例：创建技能资产只需几行代码
[Name("角色技能")]
public class SkillAsset : Asset {
    public string Test;
    public int EventName;
}

// 示例：创建自定义 Clip
[Name("普通粒子片段")]
[Attachable(typeof(EffectTrack))]
public class TestClip : ActionClip {
    [SerializeField] private float length = 1f;
    public override float Length {
        get => length;
        set => length = value;
    }
}
```

---

## 7. 项目成熟度与维护状态

| 维度 | Unity-ACT-framework | ActionEditor |
|------|---------------------|--------------|
| 版本号 | 无正式版本号（开发中） | 2.0.0 |
| 提交频率 | 活跃（2026年1月仍在更新） | 较低（2025年7月最后更新） |
| 文档 | README + UML 架构图 | README + 9篇操作手册文档 |
| 示例项目 | 框架本身即示例项目 | 单独的示例项目 ActionEditorExample |
| 已知问题 | 标注"框架还在施工" | 2.0版本有UI bug，核心功能基本完成 |
| 社区 | 视频教程（B站） | QQ群 (567604178) + 视频预览 |
| 发布形式 | Unity 完整项目 | Unity Package Manager (UPM) 包 |
| 生产就绪 | 否（开发中） | 1.0版本有上线案例 |

---

## 8. 依赖关系对比

### Unity-ACT-framework 依赖
| 依赖项 | 是否必须 | 说明 |
|--------|---------|------|
| Unity Timeline | ✅ 必须 | 核心动画和动作编排 |
| Unity Input System | ✅ 必须 | 现代化输入处理 |
| Cinemachine | 推荐 | 相机系统 |
| NodeCanvas | 可选（付费） | AI 行为树系统 |

### ActionEditor 依赖
| 依赖项 | 是否必须 | 说明 |
|--------|---------|------|
| FullSerializer | ✅ 内置 | JSON 序列化（内置于 Runtime/ThirdParty） |
| 无其他依赖 | — | 纯编辑器工具，零外部依赖 |

---

## 9. 适用场景分析

### 9.1 选择 Unity-ACT-framework 的场景

- ✅ 正在开发 **3D 动作游戏**，需要完整的战斗框架
- ✅ 需要 **开箱即用** 的战斗系统原型
- ✅ 团队熟悉 Unity Timeline 系统
- ✅ 需要 **锁定目标** 和 **多方向攻击** 等 ACT 游戏标配功能
- ✅ 希望学习完整 ACT 框架的 **架构设计思路**
- ❌ 不适合需要大规模自定义编辑工具的场景
- ❌ 不适合非动作类游戏项目

### 9.2 选择 ActionEditor 的场景

- ✅ 需要为项目构建 **自定义技能编辑器**
- ✅ 需要 **Buff/场景/剧情** 等各类时间轴编辑工具
- ✅ 项目有 **多种不同类型** 的时间轴编辑需求
- ✅ 希望 **完全控制** 编辑器的外观和行为
- ✅ 需要 **版本控制友好** 的 JSON 数据格式
- ✅ 追求 **零依赖** 的轻量级工具
- ❌ 不提供任何游戏运行时逻辑
- ❌ 需要自行实现运行时播放和业务逻辑

---

## 10. 互补性分析

两个项目实际上具有很强的 **互补性**：

```
ActionEditor (编辑工具层)
    ↕ 可以替代/增强
Unity-ACT-framework 中的 Timeline 编辑体验
```

**理论上的集成方案**：

1. **使用 ActionEditor 替代 Unity Timeline 编辑器**：将 Unity-ACT-framework 中的 ActionTimelineAsset 配置迁移到 ActionEditor 的 Asset-Group-Track-Clip 数据模型，获得更灵活的编辑体验。

2. **在 ActionEditor 中实现 ACT 业务逻辑**：继承 ActionEditor 的基类，实现攻击判定片段、移动控制片段、特效播放片段等，将 Unity-ACT-framework 的游戏逻辑与 ActionEditor 的编辑能力结合。

3. **各取所长**：使用 Unity-ACT-framework 的运行时架构（Actor、移动、相机、输入等），但用 ActionEditor 替换技能配置和编辑部分。

---

## 11. 总结

| 对比维度 | Unity-ACT-framework | ActionEditor |
|---------|---------------------|--------------|
| **项目性质** | 完整游戏框架 | 编辑器工具 |
| **关注层面** | 运行时游戏逻辑 | 编辑时数据配置 |
| **Timeline 方案** | Unity 原生 Timeline | 自研数据模型 |
| **开箱即用程度** | 高（直接可用的战斗框架） | 低（需要自行扩展业务逻辑） |
| **扩展灵活性** | 中等 | 极高 |
| **学习成本** | 较高（需理解多个游戏子系统） | 较低（只需理解数据模型和继承） |
| **适用项目类型** | 3D 动作游戏 | 任何需要时间轴编辑的项目 |
| **外部依赖** | 较多（Timeline、InputSystem、Cinemachine） | 极少（仅内置 FullSerializer） |
| **数据格式** | Unity 原生资产 | JSON 文本 |
| **是否可独立使用** | 是（完整项目） | 是（UPM包） |

**核心区别总结**：

Unity-ACT-framework 是一个 **"纵深型"** 项目——深入动作游戏领域，提供从输入到战斗到相机的完整解决方案。它回答的问题是"如何用 Unity 构建一个动作游戏"。

ActionEditor 是一个 **"横向型"** 项目——在编辑器工具层面提供通用的时间轴编辑能力，可以适配各种不同类型的项目需求。它回答的问题是"如何为任意项目构建一个时间轴编辑工具"。

两者的本质差异在于 **抽象层次不同**：Unity-ACT-framework 工作在具体的游戏逻辑层面，而 ActionEditor 工作在更底层的工具/框架层面。这也决定了它们面向的用户群体和使用方式截然不同，但同时也意味着它们有很好的互补和集成潜力。
