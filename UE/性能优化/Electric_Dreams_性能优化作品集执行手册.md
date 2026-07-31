# Electric Dreams 性能优化作品集执行手册

> 目标：把 Unreal Engine 官方 Electric Dreams 样例改造成一个“可测量、可复现、可解释”的性能优化项目，并最终整理成技术美术性能优化作品集。

---

## 目录

1. [项目定位与最终成果](#1-项目定位与最终成果)
2. [开始前必须确定的目标](#2-开始前必须确定的目标)
3. [项目安装、备份与版本管理](#3-项目安装备份与版本管理)
4. [建立可重复的基准测试](#4-建立可重复的基准测试)
5. [采集 Baseline 数据](#5-采集-baseline-数据)
6. [判断 CPU、GPU 或内存瓶颈](#6-判断-cpugpu-或内存瓶颈)
7. [第一阶段：World Partition Streaming](#7-第一阶段world-partition-streaming)
8. [第二阶段：HLOD](#8-第二阶段hlod)
9. [第三阶段：PCG 生成与实例优化](#9-第三阶段pcg-生成与实例优化)
10. [第四阶段：Actor、组件、碰撞与导航](#10-第四阶段actor组件碰撞与导航)
11. [第五阶段：Nanite 与几何体](#11-第五阶段nanite-与几何体)
12. [第六阶段：植被专项优化](#12-第六阶段植被专项优化)
13. [第七阶段：材质与 Shader](#13-第七阶段材质与-shader)
14. [第八阶段：纹理与显存](#14-第八阶段纹理与显存)
15. [第九阶段：Lumen](#15-第九阶段lumen)
16. [第十阶段：Virtual Shadow Maps](#16-第十阶段virtual-shadow-maps)
17. [第十一阶段：Niagara、流体与后处理](#17-第十一阶段niagara流体与后处理)
18. [第十二阶段：运行时卡顿和加载](#18-第十二阶段运行时卡顿和加载)
19. [建立画质等级与设备配置](#19-建立画质等级与设备配置)
20. [最终回归测试](#20-最终回归测试)
21. [作品集页面结构](#21-作品集页面结构)
22. [推荐视频脚本](#22-推荐视频脚本)
23. [四周执行计划](#23-四周执行计划)
24. [命令与视图速查表](#24-命令与视图速查表)
25. [最终检查清单](#25-最终检查清单)
26. [官方参考资料](#26-官方参考资料)

---

# 1. 项目定位与最终成果

## 1.1 推荐作品集标题

中文：

```text
Electric Dreams：4×4 km PCG 开放世界性能与可扩展性优化
```

英文：

```text
Electric Dreams Scalability Pass
Performance Optimization for a 4×4 km PCG Open World
```

## 1.2 推荐项目描述

```text
以 Unreal Engine 官方 Electric Dreams 环境为基础，针对指定中端 PC 硬件建立
16.67 ms 帧预算。通过 World Partition Streaming、HLOD、PCG 分层生成、
Nanite/植被 Overdraw、材质、Lumen、Virtual Shadow Maps 和纹理流送优化，
在尽量保持原始视觉质量的前提下，提高稳定帧率、降低内存占用并减少加载卡顿。
```

## 1.3 最终需要交付的内容

建议最终准备以下成果：

```text
01_项目说明
├── 项目目标
├── 目标硬件
├── 引擎版本
└── 性能预算

02_数据
├── Baseline 数据
├── Optimized 数据
├── CSV 帧数据
├── Unreal Insights 截图
└── ProfileGPU 截图

03_视觉证据
├── Before / After
├── Shader Complexity
├── Nanite Overdraw
├── Lumen Surface Cache
├── VSM Cache
└── World Partition Cell

04_作品集
├── 一篇完整案例文章
├── 2～4 分钟讲解视频
├── 一页总结图
└── 可选：可运行打包版本
```

## 1.4 这个项目真正要证明的能力

你要证明的不是“会输入控制台命令”，而是：

- 能建立准确的测试条件；
- 能定位 CPU、GPU、内存和 IO 瓶颈；
- 能阅读 Unreal Insights 和 GPU Profile；
- 能理解 PCG、Nanite、Lumen、VSM 的协作关系；
- 能在画质、性能、内存之间做取舍；
- 能用数据证明每一次修改有效；
- 能发现优化带来的副作用并完成回归测试。

---

# 2. 开始前必须确定的目标

没有目标硬件，就没有真正意义上的优化。

## 2.1 填写目标硬件

复制下面模板并填写：

```text
操作系统：Windows 11
CPU：________________________
GPU：________________________
显存：_______________________
系统内存：___________________
硬盘：SSD / NVMe
显示分辨率：_________________
目标输出分辨率：_____________
内部渲染比例：_______________
目标帧率：60 FPS
目标帧时间：16.67 ms
```

如果没有专门的中端测试机，就使用自己的电脑，但必须在作品集中写清楚实际硬件，不能只写“提升到 60 FPS”。

## 2.2 建议性能预算

60 FPS 的完整帧预算是：

```text
1000 / 60 = 16.67 ms
```

可以先采用以下项目目标：

| 项目 | 建议目标 |
|---|---:|
| 总 Frame Time P95 | ≤ 16.67 ms |
| GPU P95 | ≤ 15.5 ms |
| Game Thread P95 | ≤ 10～12 ms |
| Render Thread P95 | ≤ 10～12 ms |
| 超过 33.3 ms 的帧 | 尽量为 0 |
| VRAM 峰值 | 不超过实际显存的 85%～90% |
| Texture Streaming | 不持续 Over Budget |
| 60 秒路线严重 Pop-in | 0 |
| 60 秒路线明显加载停顿 | 0 |

这些数值不是引擎硬性标准，而是你为作品集建立的项目约束。

## 2.3 固定画质条件

决定并记录：

- Lumen GI：开启；
- Lumen Reflections：开启或使用 SSR，必须写明；
- Hardware Ray Tracing：开启或关闭，必须固定；
- Virtual Shadow Maps：开启；
- Nanite：开启；
- TSR：开启；
- 输出分辨率固定；
- `r.ScreenPercentage` 固定；
- Scalability 固定。

如果最后降低了 Screen Percentage，也必须单独报告，不能把它伪装成内容优化。

---

# 3. 项目安装、备份与版本管理

## 3.1 安装 Electric Dreams

1. 打开 Fab 或 Epic Games Launcher；
2. 搜索 `Electric Dreams Environment`；
3. 添加到库；
4. 在 Fab Library 中选择与样例兼容的 Unreal Engine 版本；
5. 创建项目；
6. 将项目放在 SSD 或 NVMe 硬盘；
7. 第一次打开后等待 Shader 编译完成；
8. 不要在 Shader 尚未编译完成时采集性能。

## 3.2 确认测试关卡

样例中最重要的几个关卡：

```text
/Game/Levels/ElectricDreams_Env
    完整环境，包含手工区域、PCG、Soundscape、流体和演示 Sequence。

/Game/Levels/PCG/ElectricDreams_PCG
    完整 PCG 版本。

/Game/Levels/PCG/ElectricDreams_PCGCloseRange
    较小、较友好的近距离拆解场景。
```

作品集主测试建议使用：

```text
/Game/Levels/ElectricDreams_Env
```

`ElectricDreams_PCGCloseRange` 可以作为单项 PCG 或材质实验关卡，但不能代替完整场景最终数据。

## 3.3 复制原始关卡

在 Content Browser 中：

1. 找到 `ElectricDreams_Env`；
2. 右键选择 `Duplicate`；
3. 命名为：

```text
ED_Baseline
```

4. 再复制一次；
5. 命名为：

```text
ED_Optimized
```

建议创建自己的目录：

```text
/Game/PortfolioOptimization/
├── Maps/
├── PCG/
├── Materials/
├── HLOD/
├── Sequences/
├── Data/
└── Debug/
```

不要覆盖官方资产。需要改材质、PCG Graph、Mesh 或蓝图时，先复制到自己的目录，再替换引用。

## 3.4 使用版本控制

至少建立以下版本节点：

```text
00_Original
01_BaselineCaptured
02_WorldPartition
03_HLOD
04_PCG
05_NaniteAndFoliage
06_Materials
07_LumenAndVSM
08_MemoryAndStreaming
09_Final
```

如果不使用 Git 或 Perforce，至少每天复制一个带日期的项目备份。

---

# 4. 建立可重复的基准测试

## 4.1 为什么不能随便在视口走一圈

下面这些差异都会让数据失效：

- 摄像机方向不同；
- 看到的树木数量不同；
- Shader 尚未编译；
- Streaming 尚未稳定；
- 录屏软件运行状态不同；
- Editor 窗口分辨率不同；
- Scalability 自动变化；
- 一个版本使用 100% Screen Percentage，另一个使用 50%。

因此必须制作固定摄像机路线。

## 4.2 使用官方 Fly-Through

Electric Dreams 中可以尝试：

```text
Shift + C：Fly-Through
Shift + V：PCG Mid Range
Shift + B：PCG Long Range
Spacebar：停止 Sequence
```

先确认这些快捷键在你的版本中仍然有效。

优点：

- 不需要重新制作路线；
- 路线覆盖官方展示区域；
- 容易重复。

缺点：

- 路线不一定覆盖你需要重点测试的所有区域；
- 可能包含镜头切换，对 Streaming 和 VSM Cache 造成特殊压力。

## 4.3 制作自己的 Benchmark Sequence

推荐为作品集另外制作一条路线。

### 操作

1. 打开 `ED_Baseline`；
2. 在顶部菜单选择 Cinematics；
3. 选择 `Add Level Sequence`；
4. 保存到：

```text
/Game/PortfolioOptimization/Sequences/LS_Benchmark
```

5. 在关卡中创建 Cine Camera Actor；
6. 将 Camera 添加到 Sequencer；
7. 添加 Transform Track；
8. 设置 60～90 秒路线；
9. 设置 Camera Cuts Track；
10. 尽量使用连续飞行，减少不必要的硬切镜头；
11. 确保路线包含以下区域：

```text
0～15 秒：森林近景和树叶 Overdraw
15～30 秒：河床和大量 PCG 碎片
30～45 秒：大型悬崖 Assembly
45～60 秒：快速移动和远景
可选 60～90 秒：水洼、流体、复杂材质、局部灯光
```

12. 将同一个 Sequence 用于 Baseline 和 Optimized 地图；
13. 不要分别制作两条近似路线。

## 4.4 固定运行条件

在正式采集前输入：

```text
r.VSync 0
t.MaxFPS 0
r.DynamicRes.OperationMode 0
```

然后固定：

```text
r.ScreenPercentage 100
```

如果目标明确使用 TSR 内部低分辨率，也可以改为其他固定值，例如：

```text
r.ScreenPercentage 67
```

但从 Baseline 到最终版本都必须保持一致。

## 4.5 使用打包版本测试

不要只使用 Editor Viewport FPS。

推荐：

1. 打开 Project Settings；
2. 检查 Maps & Modes；
3. 设置 Benchmark 测试地图；
4. 打开 Platforms 或 Project Launcher；
5. 打包 Development 或 Test 版本；
6. 确保目标地图被 Cook；
7. 在独立程序中运行测试。

Development 方便使用调试命令；Test 更接近 Shipping 性能，但仍保留部分分析能力。

## 4.6 每次测试流程

严格执行：

```text
1. 重启打包程序
2. 进入测试关卡
3. 等待 30～60 秒，让纹理和 Nanite Streaming 稳定
4. 播放一次测试路线作为预热，不记录
5. 播放第二次，记录 Run 1
6. 播放第三次，记录 Run 2
7. 播放第四次，记录 Run 3
8. 使用三次结果中位数
```

另外再做一次冷启动测试，专门统计加载卡顿。不要把冷启动和热运行数据混在一起。

---

# 5. 采集 Baseline 数据

## 5.1 快速查看线程瓶颈

输入：

```text
stat unit
stat unitgraph
```

记录：

```text
Frame：总帧时间
Game：Game Thread
Draw：Render Thread
GPU：GPU 帧时间
```

判断原则：

- GPU 最大并长期接近 Frame：通常 GPU Bound；
- Game 最大：Game Thread Bound；
- Draw 最大：Render Thread Bound；
- 大部分时间正常，但偶尔几十毫秒：Hitch，需要查加载、PCG、PSO 或资源流送。

注意：这些线程并行工作，不能简单把 Game、Draw、GPU 三个数字相加。

## 5.2 记录 GPU Pass

输入：

```text
stat gpu
profilegpu
```

保存 ProfileGPU 截图，重点记录：

```text
PrePass
BasePass
Nanite
ShadowDepths / VirtualShadowMaps
Lumen Scene Lighting
Lumen Screen Probe Gather
Lumen Reflections
Translucency
PostProcessing
TSR
Volumetric Fog
```

每个版本都在同一个相机位置捕获 GPU Profile。

## 5.3 记录渲染规模

输入：

```text
stat rhi
stat scenerendering
```

记录：

- Draw Primitive Calls；
- Draw Calls；
- 三角形或 Primitive 数量；
- Dynamic Mesh Elements；
- Visible Lights；
- Shadowed Lights；
- 实例数量。

不同版本中的 Stat 名称可能略有变化，记录你实际看到的字段即可。

## 5.4 记录纹理流送

输入：

```text
stat streaming
```

记录：

- Texture Pool；
- Wanted Pool；
- Required Pool；
- Visible Mips；
- Hidden Mips；
- Over Budget；
- Streaming Update 时间。

如果持续出现红色 Over Budget，先查纹理使用，不要立刻增加 Pool Size。

## 5.5 使用 Unreal Insights

### Editor/打包版本操作思路

不同 UE 版本的入口可能位于 Tools、Window > Developer Tools 或底部 Trace 菜单。

需要打开的常用 Trace Channel：

```text
CPU
GPU
Frame
Bookmark
Asset Load / Load Time
File
Memory（需要时单独测试）
```

对内存进行完整追踪时，可以通过启动参数运行：

```text
-trace=default,memory
```

若要获得更清晰的 CPU Scope 名称，可添加：

```text
-statnamedevents
```

在 Timing Insights 中重点查看：

```text
Game Thread
Render Thread
RHI Thread
Task Graph
Async Loading Thread
UPCGSubsystem::Tick
World Partition 加载
Shader / PSO 相关停顿
```

## 5.6 使用 CSV Profiler

控制台输入：

```text
csvprofile start
```

播放完整测试路线，然后输入：

```text
csvprofile stop
```

CSV 通常保存在项目的：

```text
Saved/Profiling/CSV/
```

如果版本支持固定帧数，可以使用：

```text
csvprofile frames=3600
```

60 FPS 下 3600 帧约等于 60 秒。

## 5.7 Baseline 数据表

```markdown
| 指标 | Run 1 | Run 2 | Run 3 | 中位数 |
|---|---:|---:|---:|---:|
| 平均 FPS | | | | |
| 1% Low FPS | | | | |
| 平均 Frame Time | | | | |
| P95 Frame Time | | | | |
| Game Thread P95 | | | | |
| Render Thread P95 | | | | |
| GPU P95 | | | | |
| Draw Calls | | | | |
| VRAM 峰值 | | | | |
| RAM 峰值 | | | | |
| Texture Pool | | | | |
| >33.3 ms 帧数 | | | | |
```

## 5.8 建立优化日志

每一项修改都使用以下模板：

```markdown
## OPT-001：优化标题

- 日期：
- 测试版本：
- 测试位置：
- 现象：
- 分析证据：
- 假设根因：
- 实际修改：
- 修改前数据：
- 修改后数据：
- 性能变化：
- 视觉变化：
- 内存变化：
- 是否保留：是 / 否
- 后续问题：
```

一次只改一个系统，或者一组逻辑明确、可以单独回滚的修改。

---

# 6. 判断 CPU、GPU 或内存瓶颈

## 6.1 GPU Bound

特征：

- `stat unit` 中 GPU 明显最高；
- 降低分辨率后 FPS 明显提升；
- ProfileGPU 中某个 Pass 耗时突出。

优先检查：

```text
Lumen
Virtual Shadow Maps
Masked Foliage
Nanite Overdraw
Translucency
材质复杂度
TSR
体积雾
后处理
```

## 6.2 Game Thread Bound

特征：

- Game Thread 最高；
- 降低分辨率没有明显变化；
- Insights 中大量 Tick、PCG、碰撞、蓝图或对象生成。

优先检查：

```text
Runtime PCG
Actor 数量
组件数量
Tick
碰撞和 Overlap
导航
物理
Soundscape
蓝图逻辑
```

## 6.3 Render Thread Bound

特征：

- Draw/Render Thread 最高；
- Draw Call 和 Primitive 数量大；
- 大量独立 Actor 和独立材质状态。

优先检查：

```text
ISM/HISM
PCG Static Mesh Spawner
独立 MID
材质槽数量
非 Nanite Mesh
阴影提交
Scene Capture
```

## 6.4 内存或流送问题

特征：

- 平均帧正常，但移动时卡顿；
- `stat streaming` 经常 Over Budget；
- VRAM 接近物理上限；
- Nanite 或纹理反复 Stream In/Out；
- World Partition Cell 加载时出现几十毫秒停顿。

优先检查：

```text
World Partition Streaming
HLOD
纹理尺寸和 Mip
Nanite Streaming Pool
强制常驻资源
Actor 交叉引用
PCG Runtime Generation
PSO / Shader
```

---

# 7. 第一阶段：World Partition Streaming

Electric Dreams 完整关卡约为 4×4 km，并且官方样例默认关闭 World Partition Streaming。这个阶段通常是最有作品集价值的第一项修改。

## 7.1 开启 Streaming

在 `ED_Optimized` 中：

1. 打开关卡；
2. 打开 `World Settings`；
3. 找到 World Partition 或 World Partition Setup；
4. 找到 `Enable Streaming`；
5. 开启；
6. 保存关卡；
7. 重新打开关卡；
8. 检查是否出现丢失引用或始终加载的大型对象组。

不同 UE 小版本的分类名称可能略有变化，但核心开关是 `Enable Streaming`。

## 7.2 检查 Runtime Grid

在 World Settings 的 World Partition Runtime Settings 中检查：

- Runtime Grid；
- Cell Size；
- Loading Range；
- Block on Slow Streaming；
- HLOD Layer；
- Streaming Source。

不要先照抄一个“最佳数值”。建立测试组：

```text
方案 A：较大 Cell + 较大 Loading Range
方案 B：中等 Cell + 中等 Loading Range
方案 C：较小 Cell + 较小 Loading Range
```

分别记录：

- 常驻内存；
- 峰值内存；
- Cell 加载时间；
- Actor 初始化峰值；
- 快速飞行时 Pop-in；
- HLOD 切换；
- 60 秒路线 Hitch 数量。

判断原则：

- Cell 太大：一次加载内容太多，内存和加载峰值大；
- Cell 太小：Cell 数量和调度开销大，频繁加载卸载；
- Loading Range 太小：Pop-in 和快速移动加载不及时；
- Loading Range 太大：内存回升，失去 Streaming 优势。

## 7.3 检查 Is Spatially Loaded

在 Outliner 中分组检查：

```text
大型岩石 Assembly
树木和森林
河床碎片
局部装饰
灯光
音频
流体
演示用 Actor
全局系统
```

选择 Actor，在 Details > World Partition 检查：

```text
Is Spatially Loaded
Runtime Grid
HLOD Layer
```

建议：

| Actor 类型 | Is Spatially Loaded |
|---|---|
| 普通树木、岩石、装饰 | 开启 |
| 区域局部灯光和音频 | 通常开启 |
| GameMode 或全局管理器 | 关闭 |
| 必须始终存在的系统 | 关闭 |
| 仅编辑器说明 Actor | 打包时移除或不参与运行 |

## 7.4 查找交叉引用导致的捆绑加载

World Partition 中，一个 Spatially Loaded Actor 如果硬引用一个 Always Loaded Actor，或者多个 Actor 形成相互硬引用，可能导致它们一起加载。

操作思路：

1. 在 World Partition Cell 加载后观察实际加载 Actor；
2. 检查异常常驻的大型 Actor；
3. 在 Reference Viewer 中查看硬引用；
4. 能使用软引用时改为 Soft Object Reference；
5. 不要让一个全局蓝图持有整张地图所有装饰物的硬引用；
6. Sequence、演示控制器、测试对象也要检查。

## 7.5 配置 Streaming Source

默认玩家控制器通常可以作为 Streaming Source。

对于高速 Drone：

1. 检查 Player Controller 是否启用 Streaming Source；
2. 如果摄像机和 Pawn 距离很远，在摄像机或路线前方添加 `World Partition Streaming Source Component`；
3. 设置合理 Loading Range；
4. 提前加载即将进入的区域；
5. 离开后停用额外 Streaming Source；
6. 避免永久开启多个大范围 Streaming Source。

## 7.6 World Partition 阶段测试

至少测试：

- 静止 60 秒；
- 慢速飞行；
- 高速飞行；
- 突然转身；
- Sequence 硬切镜头；
- 从森林进入远景；
- 原地快速升高；
- 传送到远处。

## 7.7 阶段验收

通过条件：

- 完整地图不再全部常驻；
- RAM 和 VRAM 相比 Baseline 明显下降；
- 相同路线不存在严重几何缺失；
- 快速移动时没有不可接受的 Pop-in；
- World Partition 加载没有制造更严重的 P95/P99 卡顿；
- 所有差异都在日志中记录。

建议作品集截图：

```text
Baseline：Streaming Disabled，全场景驻留
Optimized：Streaming Enabled，仅附近 Cell + 远处 HLOD
```

---

# 8. 第二阶段：HLOD

World Partition 负责卸载远处 Cell，HLOD 负责在原始 Cell 卸载后继续显示低成本远景。

## 8.1 创建 HLOD Layer Asset

在 Content Browser 中：

1. 进入：

```text
/Game/PortfolioOptimization/HLOD/
```

2. 右键创建 HLOD Layer；
3. 如果右键菜单中没有，搜索 `HLOD Layer`；
4. 建议建立：

```text
HLOD_Cliffs
HLOD_ForestFar
HLOD_Rocks
HLOD_Background
```

## 8.2 选择 HLOD 类型

常见类型及用途：

```text
Instancing
    适合同一 Mesh 的重复实例。

Merged Mesh
    合并多个物体，减少 Draw Call，几何质量保持较高。

Simplified Mesh
    合并后进行简化，适合远距离大型组合。

Custom
    使用自定义 HLOD 生成逻辑。
```

建议：

| 内容 | 建议 |
|---|---|
| 大型悬崖和岩石 Assembly | Simplified Mesh 或 Merged Mesh |
| 同类远处树木 | Instancing 或专门远景方案 |
| 小碎片群 | 合并、简化或直接远距离剔除 |
| 重要轮廓背景 | Simplified Mesh，保留轮廓 |

## 8.3 分配 HLOD Layer

可以：

- 在 World Settings 设置默认 HLOD Layer；
- 在 Actor 或组件 Details 中单独设置；
- 在 Data Layer/PCG 输出中设置对应 HLOD Layer；
- 给不同内容建立不同 HLOD 策略。

不要让所有资产都使用完全相同的 HLOD 设置。

## 8.4 构建 HLOD

1. 保存所有关卡和资产；
2. 找到 Build 菜单；
3. 选择 Build HLODs 或 World Partition HLODs；
4. 等待构建完成；
5. 检查日志错误；
6. 检查生成资产体积；
7. 检查代理材质数量；
8. 检查远景轮廓；
9. 在运行时观察 Cell 卸载和 HLOD 切换。

## 8.5 重点检查的问题

- HLOD 材质过多；
- 代理纹理分辨率过高；
- 透明或 Masked 材质合并错误；
- Nanite Mesh 的 HLOD 收益不明显；
- 远景轮廓破坏；
- 切换时跳变；
- HLOD 自身仍有过多材质槽；
- HLOD 纹理占用抵消内存收益。

## 8.6 阶段验收

- 远处 Cell 卸载后背景仍然完整；
- Draw Call 和常驻 Actor 数量下降；
- HLOD 资源没有造成异常纹理内存增长；
- 常规镜头下切换不明显；
- 快速移动时没有长时间空洞；
- 记录 HLOD 构建时间与打包体积变化。

---

# 9. 第三阶段：PCG 生成与实例优化

## 9.1 先决定是否需要运行时生成

对每个 PCG Graph 分类：

```text
A. 永远不会变化
    编辑器中生成并保存，不需要运行时重新生成。

B. 只在关卡加载时变化一次
    可以生成一次并缓存，避免每帧或重复生成。

C. 随玩家位置变化
    使用 Runtime Generation + Hierarchical Generation。

D. 真正交互式
    只让必要的小范围内容动态更新。
```

作品集里写清楚：为什么某组内容选择 Bake，为什么另一组保留 Runtime PCG。

## 9.2 定位 PCG Graph

1. 在 Outliner 搜索 `PCG`；
2. 选择 PCG Volume 或带 PCG Component 的 Actor；
3. 在 Details 找到绑定的 PCG Graph；
4. 双击打开 Graph；
5. 在 Debug Object Tree 选择实际调试对象；
6. 不要只打开 Graph 而没有选择 Debug Object，否则看到的数据可能不是目标实例。

## 9.3 使用节点 Debug

常用操作：

```text
D：切换节点 Debug Rendering
E：启用/禁用节点
A：Inspect 节点数据（版本快捷键可能变化）
```

对每个关键节点记录：

- 输入点数量；
- 输出点数量；
- Metadata 属性数量；
- 节点 CPU 时间；
- 是否缓存；
- 是否重复执行；
- 是否产生大量无用点。

## 9.4 打开 PCG Profiling

在 PCG Graph Editor 中找到 Profiling 面板。如果当前布局没有显示：

1. 查看 Window 菜单；
2. 打开 Profiling；
3. 选择目标 Debug Object；
4. 强制重新生成；
5. 记录节点耗时；
6. 如果节点结果被缓存，先 Flush Cache 或 Ctrl/修饰键触发 Force Regen；
7. 分别记录首次生成和缓存命中的结果。

重点查找：

```text
Surface Projection
Ray Hit Query
Copy Points
Density Noise
Attribute Math
Transform Points
复杂 Subgraph
Static Mesh Spawner
循环或重复采样
```

## 9.5 尽早过滤点

错误顺序：

```text
生成 1,000,000 个点
→ 为全部点计算十几个 Attribute
→ Surface Project
→ 复杂随机和旋转
→ 最后删除 90%
```

优化顺序：

```text
生成点
→ Bounds Filter
→ Density Filter
→ Slope / Height Filter
→ Distance Filter
→ 剩余点才做投射和复杂 Attribute
→ Spawn
```

操作：

1. 在复杂节点前记录点数；
2. 把空间过滤节点移动到前面；
3. 确认视觉分布没有改变；
4. 再次 Profile；
5. 记录节点时间和总生成时间。

## 9.6 优化 Copy Points

Electric Dreams 官方 PCG 流程中，若不需要继承所有属性，`Copy Points` 可以将 Attribute Inheritance 设置为：

```text
Source Only
```

操作：

1. 选择 Copy Points 节点；
2. 在 Details 找到 Attribute Inheritance；
3. 检查后续节点真正使用哪些 Attribute；
4. 若只需源数据，选择 Source Only；
5. 删除不再使用的中间 Metadata；
6. 重新生成并验证 Mesh、Transform、材质和 Tag。

## 9.7 使用 ISM/HISM

目标：避免每个树、石头、树枝都成为独立 Actor。

检查 Static Mesh Spawner：

1. 选择 Spawner；
2. 检查输出是否使用实例组件；
3. 检查 Mesh Selector；
4. 检查是否因为每实例唯一材质而破坏实例化；
5. 检查是否生成独立 Actor；
6. 同一 Mesh、同一材质尽量进入相同实例组；
7. 需要每实例变化时使用 Per Instance Custom Data，而不是大量独立 MID。

## 9.8 Runtime Generation

只有确认需要运行时生成时再开启。

操作方向：

1. 打开 PCG Component；
2. 将 Generation Trigger/Generation Mode 设置为 Runtime；
3. 开启 Partitioned 或适合 World Partition 的分区模式；
4. 在 Graph Settings 配置 Hierarchical Generation；
5. 设置不同 Grid Size；
6. 为大尺度内容设置较大生成半径；
7. 为细节内容设置较小生成半径；
8. 设置 Cleanup Radius Multiplier；
9. 测试快速移动；
10. 观察生成和清理是否频繁抖动。

常用调试命令：

```text
pcg.RuntimeGeneration.Enable
pcg.RuntimeGeneration.EnableDebugging
pcg.RuntimeGeneration.EnablePooling
pcg.RuntimeGeneration.NumGeneratingComponents
pcg.FrameTime
```

不要把增加 `pcg.FrameTime` 当成优化。它只是允许 PCG 每帧使用更多时间，可能让生成更快，但也可能降低帧率。

## 9.9 PCG 内容剔除

针对每一类输出设置：

- Spawn Density；
- 最小距离；
- Max Draw Distance；
- Cull Distance；
- 是否投射阴影；
- 是否产生距离场；
- 是否影响导航；
- 是否有碰撞；
- 是否需要 WPO；
- 是否进入 HLOD。

## 9.10 PCG 阶段验收

- 静态内容不再无意义地运行时重建；
- PCG Graph 总生成时间下降；
- 复杂节点前的点数量明显减少；
- 独立 Actor/组件数量下降；
- Runtime PCG 不产生连续明显 Hitch；
- 快速移动时生成速度可接受；
- 视觉密度和构图保持；
- 每项调整有前后 Profiler 截图。

---

# 10. 第四阶段：Actor、组件、碰撞与导航

## 10.1 检查 Actor 与组件数量

使用：

```text
stat scenerendering
Unreal Insights
Outliner
```

对比：

- 总 Actor 数；
- Primitive Component 数；
- 独立 Static Mesh Component 数；
- ISM/HISM Instance 数；
- Tick 数量；
- 注册/注销组件造成的峰值。

## 10.2 关闭无意义 Tick

检查蓝图和组件：

1. 打开 Class Defaults；
2. 搜索 Tick；
3. 对静态环境 Actor 关闭 `Start with Tick Enabled`；
4. 如果只在事件发生时工作，改为事件驱动；
5. 如果低频更新即可，降低 Tick Interval；
6. 用 Insights 验证 Tick 时间是否下降。

## 10.3 碰撞优化

分类：

```text
玩家能接触的大型表面
    保留简单碰撞。

纯视觉小碎片
    No Collision。

大型复杂岩石
    使用简化碰撞，避免默认 Complex as Simple，除非确有必要。

远处 HLOD
    通常不需要碰撞。
```

操作：

1. 打开 Static Mesh Editor；
2. 查看 Collision；
3. 使用 Show > Simple Collision/Complex Collision；
4. 删除无意义复杂碰撞；
5. 生成 Box、Capsule、Convex 或自定义简单碰撞；
6. 在组件上关闭 Generate Overlap Events；
7. 设置正确 Collision Preset；
8. 对 PCG 输出使用无碰撞版本或 Spawner Override。

## 10.4 导航优化

纯装饰物通常不应影响导航。

检查组件：

```text
Can Ever Affect Navigation
```

对小石头、树叶、枝条、苔藓和不可交互碎片关闭。

## 10.5 阶段验收

- Game Thread 和 Render Thread 提交成本下降；
- 组件注册峰值下降；
- 玩家仍然可以正常碰撞关键地形；
- 不出现穿过主要地面或岩石的问题；
- 不影响需要的导航结果；
- PCG 内容加载/卸载时卡顿降低。

---

# 11. 第五阶段：Nanite 与几何体

## 11.1 打开 Nanite Visualization

在 Viewport：

```text
View Mode
→ Nanite Visualization
→ Overview
```

分别查看：

```text
Mask
Triangles
Clusters
Instances
Overdraw
Evaluate WPO
Pixel Programmable
Raster Bins
Shading Bins
```

每个关键测试镜头保存截图。

## 11.2 检查未启用 Nanite 的适合资产

适合优先检查：

- 岩石；
- 悬崖；
- 树干；
- 大型高模扫描资产；
- 固定建筑或结构；
- 高面数静态装饰。

启用步骤：

1. 打开 Static Mesh；
2. 在 Details 找到 Nanite Settings；
3. 开启 Nanite Support；
4. Apply Changes；
5. 等待构建；
6. 检查材质是否兼容；
7. 检查碰撞和 Fallback Mesh；
8. 在 Nanite Mask 中确认生效；
9. 重新测量，而不是默认认为一定更快。

## 11.3 Nanite Overdraw

Nanite Overdraw 明亮区域表示多个几何层覆盖同一批像素。

重点查看：

- PCG 岩石堆叠；
- 大型悬崖 Assembly 内部；
- 大量相互穿插树枝；
- 地面上多层碎片；
- 很多极小 Mesh 紧密叠放。

优化操作：

1. 找到 Overdraw 最亮的镜头；
2. 暂时隐藏 PCG 层或 Actor 组；
3. 确认是哪组资产造成；
4. 减少完全不可见的内部几何；
5. 降低 PCG 堆叠密度；
6. 删除重叠但不影响轮廓的层；
7. 远距离使用 HLOD；
8. 重新对比 Overdraw 和 GPU Nanite Pass。

不要只降低三角形数量。Nanite Overdraw 的关键常常是屏幕像素上的几何层数。

## 11.4 非 Nanite Mesh

对不使用 Nanite 的资产：

1. 打开 Static Mesh Editor；
2. 配置 LOD0、LOD1、LOD2……；
3. 检查 Screen Size；
4. 设置 Cull Distance；
5. 关闭远距离 WPO；
6. 检查阴影 LOD；
7. 检查材质槽数量；
8. 检查简单碰撞。

## 11.5 硬边法线与顶点数量

高面数 Mesh 如果存在大量硬边、UV Seam 和不共享顶点，实际顶点数量可能远高于建模软件中看到的点数。

检查：

- 不必要的硬边；
- 每个三角形独立法线；
- 极度碎片化 UV；
- 重复顶点；
- 不必要材质 ID。

## 11.6 阶段验收

- Nanite Mask 符合预期；
- 重点镜头 Overdraw 明显降低；
- 非 Nanite 资产具备合理 LOD；
- Nanite Streaming 不发生持续 Thrashing；
- GPU Nanite Pass 和 Render Thread 数据有改善；
- 远景轮廓没有明显破坏。

---

# 12. 第六阶段：植被专项优化

植被不是单一问题，而是以下成本叠加：

```text
大量实例
+ Masked Overdraw
+ Two Sided
+ WPO 风动画
+ VSM 阴影缓存失效
+ Lumen / Distance Field
+ 纹理采样
+ 远距离阴影
```

## 12.1 找到植被瓶颈区域

在同一个镜头依次观察：

```text
Shader Complexity
Quad Overdraw
Nanite Overdraw
Light Complexity
Virtual Shadow Map Cache
Lumen Performance Overview
```

然后依次临时关闭：

```text
植被可见性
植被阴影
WPO
Lumen Reflections
```

每次只关闭一项，确认真正的成本来源。

## 12.2 Masked Overdraw

操作：

1. 打开 Quad Overdraw；
2. 靠近树冠和草地；
3. 找到卡片层叠最严重的位置；
4. 检查叶片纹理中透明空白区域比例；
5. 减少过大的透明卡片；
6. 减少多层完全重叠的叶片卡；
7. 中远距离使用更紧凑的叶片形状；
8. 检查 Opacity Mask Clip Value；
9. 确保没有误用 Translucent；
10. 对比视觉密度和 BasePass 时间。

## 12.3 WPO 距离

远处树叶的风动画通常不可见，却可能造成：

- 顶点计算；
- Nanite WPO 成本；
- VSM Cache 失效；
- Lumen/距离场更新问题。

在材质中增加距离衰减：

```text
Camera Distance
→ SmoothStep / Saturate
→ 乘到 WPO 强度
```

概念：

```hlsl
WindFade = 1 - saturate((CameraDistance - StartFade) / FadeRange);
FinalWPO = WindOffset * WindFade;
```

操作：

1. 复制植被 Master Material；
2. 找到 WPO 输出；
3. 添加距离衰减；
4. 在 Material Instance 暴露 StartFade 和 EndFade；
5. 从较远距离开始关闭；
6. 观察树冠是否突然停止；
7. 增大过渡区避免跳变；
8. 对比 VSM Cache 和 GPU 时间。

## 12.4 阴影策略

按类型分类：

```text
主树干和大型树冠
    保留阴影。

中型灌木
    限制阴影距离。

小草、落叶、小枝条
    视目标画质关闭远距离阴影或全部动态阴影。
```

操作：

1. 在 Foliage Type、PCG Spawner 或组件上检查 Cast Shadow；
2. 检查 Dynamic Shadow；
3. 检查 Contact Shadow；
4. 设置 Cull Distance；
5. 设置阴影可见距离；
6. 从最小、最多的植被开始关闭；
7. 重新测量 VSM；
8. 检查中午、侧光、逆光三种情况。

## 12.5 植被密度分层

通过 PCG 或 Foliage Scalability 建立：

```text
近距离：完整密度
中距离：减少小型装饰
远距离：只保留轮廓树木和 HLOD
```

优先剔除对轮廓贡献很低的：

- 小叶片；
- 小石头；
- 小树枝；
- 地表碎片；
- 远处草丛。

## 12.6 阶段验收

- 森林重点镜头 BasePass/VSM 时间下降；
- Quad Overdraw 降低；
- 远距离 WPO 不再持续破坏阴影缓存；
- 小植被阴影成本降低；
- 静态截图和运动视频中仍保持森林密度；
- 没有明显的密度分界线。

---

# 13. 第七阶段：材质与 Shader

## 13.1 使用 Shader Complexity

Viewport：

```text
View Mode
→ Optimization Viewmodes
→ Shader Complexity
```

另查看：

```text
Quad Overdraw
Shader Complexity with Quad Overdraw
```

记录红色、粉色或白色高成本区域。

## 13.2 使用 Material Editor Stats

对重点材质：

1. 双击打开 Material；
2. 打开 Stats；
3. 查看目标平台 Shader；
4. 记录：

```text
Base Pass 指令数
Vertex Shader 指令数
Texture Sample 数
Samplers
是否使用 Pixel Depth Offset
是否使用 WPO
是否 Two Sided
是否 Masked / Translucent
```

不要只比较一个“Instruction Count”。材质可能在 BasePass、DepthPass、ShadowPass、Lumen Capture 等多个 Pass 中执行。

## 13.3 Material Analyzer

操作：

1. 打开顶部 `Tools`；
2. 进入 `Audit`；
3. 打开 `Material Analyzer`；
4. 选择 Electric Dreams 的重点 Master Material；
5. 查看所有 Material Instance 后代；
6. 查找：

```text
Static Switch
Base Property Override
Static Component Mask
重复排列组合
几乎相同的 Master Material
```

7. 记录 Shader Permutation 数量；
8. 合并完全重复的 Master；
9. 把只需要运行时变化的参数改为 Scalar/Vector，而不是 Static Switch；
10. 但不要把本来可以静态裁剪的昂贵分支全部改成动态分支。

## 13.4 建立材质距离分层

对岩石、树干、地面和苔藓检查：

- Detail Normal；
- Macro Variation；
- 多层混合；
- 湿润效果；
- Pixel Depth；
- Camera Vector；
- 高成本噪声；
- Triplanar；
- 多次纹理采样。

建立距离开关或连续衰减：

```text
Near：完整细节
Mid：关闭高频 Detail 和部分混合
Far：基础颜色、法线、粗糙度
```

注意动态分支不一定能节省 GPU。优先使用可验证的 Material Quality Switch、Static Switch 画质版本或更简单的远景代理材质。

## 13.5 Substrate

Electric Dreams 包含 Substrate 展示内容。对复杂 Substrate 材质：

1. 先通过 ProfileGPU 和 Material Stats 确认它是否真正影响主要镜头；
2. 检查 BSDF 层数；
3. 检查不必要混合；
4. 为远景建立简单材质；
5. 为低画质档建立非复杂分支；
6. 不要为了作品集把展示用 Shader 完全删掉；
7. 用画质分级说明取舍。

## 13.6 使用 Material Instance

把相同 Shader 结构、只改变纹理和数值的材质组织为：

```text
Master Material
└── Material Instance
    ├── MI_Rock_A
    ├── MI_Rock_B
    └── MI_Rock_Wet
```

不要为每块石头复制一个新的 Master Material。

## 13.7 Custom Primitive Data

如果大量单独组件只需要不同颜色、湿度或溶解值：

1. 在 Material 中选择 Scalar/Vector Parameter；
2. 启用 Custom Primitive Data；
3. 分配 Primitive Data Index；
4. 在组件上写入对应 Custom Primitive Data；
5. 确认多个对象仍能共享相同材质；
6. 对比大量独立 MID 的 Draw Call 和内存。

## 13.8 Per Instance Custom Data

对于 ISM/HISM：

1. 在材质中使用 `PerInstanceCustomData` 节点；
2. 为实例设置数据索引；
3. 在 PCG/ISM 中写入每实例数值；
4. 用于随机色、湿度、苔藓量等；
5. 避免为每个实例创建单独材质。

## 13.9 纹理采样与打包

检查是否可以打包：

```text
R = Ambient Occlusion
G = Roughness
B = Metallic
A = 可选 Mask
```

注意：只有采样频率、UV 和分辨率需求一致时才适合打包。

其他检查：

- 多张完全相同 Mask；
- 远距离仍采样 Detail Normal；
- 高成本程序噪声可否烘焙；
- Shared Sampler 是否适用；
- 纹理坐标计算是否可以移动到 Customized UV/Vertex；
- 不必要的 Runtime Virtual Texture Sample。

## 13.10 阶段验收

- 重点材质指令/采样减少；
- Shader Complexity 改善；
- Shader Permutation 数减少；
- BasePass 时间下降；
- 材质实例共享增加；
- 没有明显丢失湿润、苔藓和层次效果；
- Shader 首次编译和 Cook 时间变化有记录。

---

# 14. 第八阶段：纹理与显存

## 14.1 使用 Size Map

1. 在 Content Browser 选择目标文件夹或资产；
2. 右键；
3. 选择 `Size Map`；
4. 查看最大资源；
5. 分别检查：

```text
Textures
Static Meshes
Nanite Data
Materials
HLOD
Virtual Textures
Audio
```

## 14.2 使用 Asset Audit

1. 打开 `Tools`；
2. 进入 `Audit`；
3. 打开 Asset Audit；
4. 按 Disk Size 或 Resource Size 排序；
5. 找出 8K、无 Mip、Never Stream、异常压缩格式资产；
6. 导出或截图结果。

## 14.3 Texture Editor 检查项

对高占用纹理：

```text
Imported Size
Displayed Size
Max Texture Size
LOD Bias
Texture Group
Compression Settings
Mip Gen Settings
Never Stream
Virtual Texture Streaming
SRGB
```

操作原则：

- 远景纹理不需要 8K；
- Mask 通常不需要与 Base Color 一样高的分辨率；
- Detail Normal 可以使用平铺，而不是每个资产独立 8K；
- 不要关闭 Mip；
- UI 之外谨慎使用 Never Stream；
- 使用正确 Texture Group；
- 限制 Max Texture Size，而不是重新导出后丢失源文件。

## 14.4 Streaming Accuracy View

在 View Mode 中检查：

```text
Required Texture Resolution
Material Texture Scale Accuracy
Mesh UV Density Accuracy
```

如果系统错误估算某材质 UV 缩放，可能加载过高或过低 Mip。

## 14.5 Texture Pool

输入：

```text
stat streaming
```

调试时可以查看：

```text
r.Streaming.PoolSize
```

原则：

1. 先根据目标显存设预算；
2. 找出 Over Budget 原因；
3. 调整纹理大小、LOD Group 和常驻设置；
4. 最后才决定 Pool Size；
5. 不能把 Pool Size 设成无限并宣称问题解决。

## 14.6 Nanite Streaming

检查：

- 是否静止镜头下仍不断 Streaming；
- 是否出现 Cache Thrashing；
- Streaming Geometry 可视化；
- Nanite Streaming Pool；
- 大量资产是否设置不合理 Keep Resident；
- 快速飞行时 IO 和解压峰值。

不要单纯增大 Nanite Pool。增大 Pool 会减少 IO，但会增加显存或内存占用，需要对目标硬件权衡。

## 14.7 Memory Insights

使用：

```text
-trace=default,memory
```

测试点：

```text
进入关卡前
关卡加载完成
跑完一次路线
跑完三次路线
离开区域后
```

在 Memory Insights 中检查：

- Live Allocations；
- 增长但不回落的分配；
- World Partition 卸载后仍残留对象；
- PCG 生成和清理后的内存；
- LLM Tag；
- Texture、Render Target、Nanite、Audio 等类别。

## 14.8 阶段验收

- Texture Pool 不持续 Over Budget；
- VRAM 峰值低于目标预算；
- 高占用纹理都有合理用途；
- Streaming 没有持续抖动；
- HLOD 没有引入异常大纹理；
- World Partition 卸载后内存能回落；
- 近景纹理质量仍然可接受。

---

# 15. 第九阶段：Lumen

## 15.1 先记录 Lumen Pass

使用：

```text
stat gpu
profilegpu
```

记录：

```text
Lumen Scene Lighting
Lumen Screen Probe Gather
Lumen Reflections
Lumen Translucency
Surface Cache Update
```

如果 Lumen 不是主要成本，就不要为了“做过 Lumen 优化”而随意破坏画面。

## 15.2 打开 Lumen View Mode

Viewport 中查看：

```text
Lumen Overview
Lumen Scene
Surface Cache
Reflection View
Geometry Normals
Performance Overview
```

## 15.3 查看 Card Placement

输入：

```text
r.Lumen.Visualize.CardPlacement 1
```

关闭：

```text
r.Lumen.Visualize.CardPlacement 0
```

检查：

- 粉红色无 Surface Cache 覆盖区域；
- 单个 Mesh 结构是否过于复杂；
- 大型组合 Mesh 是否应该拆分；
- 材质是否依赖 Camera Vector、Pixel Depth 等视角逻辑；
- 镜面反射中是否出现黑色或错误区域。

## 15.4 修复 Surface Cache

按顺序尝试：

1. 拆分过于复杂或内部结构过多的 Mesh；
2. 确保墙、地板、顶面等具有合理独立表面；
3. 对关键 Mesh 增加 Max Lumen Mesh Cards；
4. 不要为所有小物体增加 Cards；
5. 对视角相关复杂材质使用 Ray Tracing Quality Switch；
6. 重新检查反射和 GI；
7. 记录内存和更新成本。

## 15.5 Software RT 与 Hardware RT 对照

在项目设置中检查：

```text
Rendering
→ Lumen
→ Hardware Ray Tracing
```

部分设置需要重启和 Shader 重编译。

制作两个固定测试：

```text
方案 A：Lumen Software Ray Tracing
方案 B：Lumen Hardware Ray Tracing
```

比较：

- Lumen GPU 时间；
- 反射质量；
- 植被和重叠实例成本；
- 显存；
- 快速移动稳定性；
- 目标 GPU 是否支持。

不要只根据高端开发机选择 Hardware RT。作品集需要解释为什么目标硬件采用某种模式。

## 15.6 Global Distance Field

对细小、对 GI 贡献很低的组件：

```text
Affects Distance Field Lighting = false
```

或者在 Static Mesh 中降低：

```text
Distance Field Resolution Scale
```

注意：关闭前后检查 GI、遮蔽和反射，不能批量关闭后不看结果。

静态 Actor 不要在运行时不断修改 Transform，否则会使静态距离场缓存失效。

## 15.7 Lumen Reflections

检查高光和光滑表面占屏幕比例。如果 Lumen Reflections 很贵：

1. 确认是否真的需要所有表面完整反射；
2. 检查材质 Roughness；
3. 对难以察觉的植被反射减少质量；
4. 测试 Reflection Downsample；
5. 为中档画质使用 SSR；
6. 为 High 保留完整 Lumen Reflections；
7. 做相同镜头视觉对比。

## 15.8 阶段验收

- Lumen 三个主要 Pass 有前后数据；
- Surface Cache 关键错误修复；
- 小物体 Distance Field 成本降低；
- 目标追踪方式符合目标硬件；
- GI 和反射视觉没有不可接受退化；
- 画质等级之间有清晰差别。

---

# 16. 第十阶段：Virtual Shadow Maps

## 16.1 记录 VSM 成本

使用：

```text
stat gpu
profilegpu
```

记录：

- Virtual Shadow Maps 总时间；
- Shadow Depths；
- Page Marking；
- Page Rendering；
- Non-Nanite Shadow；
- Cache 相关 Pass。

## 16.2 使用 VSM 可视化

在 View Mode 或 Virtual Shadow Map Visualization 中检查：

```text
Virtual Page
Cached Page
Invalidated Page
Clipmap / Mip
Nanite 与 Non-Nanite 阴影几何
```

具体显示名称可能随版本变化，选择能显示缓存页和失效页的模式。

## 16.3 查找缓存失效来源

常见来源：

- 灯光移动或旋转；
- 投射阴影的几何体移动；
- WPO/PDO；
- 蓝图每帧修改组件状态；
- 物体生成和销毁；
- 大范围局部光源；
- 大量 Non-Nanite 植被。

定位方法：

1. 在森林镜头捕获 VSM Cache；
2. 暂时关闭植被 WPO；
3. 再捕获；
4. 暂时关闭小植被阴影；
5. 再捕获；
6. 暂时冻结移动灯光；
7. 再捕获；
8. 找到影响最大的一项；
9. 恢复其他测试开关；
10. 只保留经过验证的正式修改。

## 16.4 优化局部灯光

对 Point、Spot、Rect Light：

1. 检查 Attenuation Radius；
2. 缩小看不到贡献的范围；
3. 对不重要灯光关闭 Cast Shadows；
4. 设置 Max Draw Distance；
5. 设置 Max Distance Fade Range；
6. 避免很多大范围阴影灯互相重叠；
7. 检查灯光 Mobility；
8. 重新 Profile。

## 16.5 Non-Nanite 阴影

对 Non-Nanite 物体：

- 配置完整 LOD；
- 远距离关闭形变；
- 设置 CPU Cull Distance；
- 小物体关闭远距离阴影；
- 适合时转换 Nanite；
- 避免无 LOD 高面数 Mesh 写入很多 VSM Page。

## 16.6 阶段验收

- VSM Cache 失效明显减少；
- 森林移动镜头 VSM 时间下降；
- 小植被不再造成大量阴影页重绘；
- 主要光照方向和层次仍然保留；
- 镜头切换时没有严重阴影缺失；
- 局部灯光优化有可量化证据。

---

# 17. 第十一阶段：Niagara、流体与后处理

## 17.1 Niagara

检查 Electric Dreams 中的环境特效、粒子和流体。

操作方向：

1. 打开 Niagara Debugger；
2. 查看 System、Emitter、Particle 数量；
3. 检查 Fixed Bounds；
4. 检查不可见时是否仍模拟；
5. 为系统设置 Effect Type；
6. 配置 Distance Culling；
7. 配置 Visibility Culling；
8. 降低远距离 Spawn Rate；
9. 降低不重要粒子更新频率；
10. 检查 GPU Simulation Stage；
11. 测量 Niagara Pass 和 Game Thread。

## 17.2 流体水洼

样例包含流体模拟展示内容。它可能是演示功能，不一定适合始终运行。

测试：

```text
流体开启
流体关闭
静止模拟
摄像机远离
流体离屏
```

如果流体只在特定展示镜头需要：

- 使用 Data Layer；
- 只在玩家靠近时激活；
- 离开后暂停；
- 降低模拟分辨率；
- 降低更新频率；
- 中低画质使用简化材质或预烘焙动画。

## 17.3 体积雾

通过 ProfileGPU 查找 Volumetric Fog。

测试：

- Fog Grid 分辨率；
- View Distance；
- 局部体积数量；
- 灯光是否影响体积雾；
- Shadowed Volumetric Lighting；
- 高、中画质分级。

## 17.4 Translucency

重点检查：

- 粒子；
- 雾片；
- 水面；
- 半透明贴花；
- 多层透明；
- 大面积屏幕覆盖。

使用 Quad Overdraw 和 ProfileGPU 确认成本。

## 17.5 后处理

逐项测试：

```text
Depth of Field
Motion Blur
Bloom
Chromatic Aberration
Vignette
SSR / Lumen Reflections
Ambient Occlusion
Color Grading
```

原则：只修改 ProfileGPU 中有明显成本的 Pass。

## 17.6 阶段验收

- 粒子离屏后正确 Cull；
- 流体不再无条件全程模拟；
- 体积和透明 Pass 有明确数据；
- 中低画质有合理替代；
- 环境氛围没有被简单删除。

---

# 18. 第十二阶段：运行时卡顿和加载

平均 FPS 达标并不代表项目完成。

## 18.1 统计 Hitch

在 60 秒测试路线中记录：

```text
>16.67 ms 帧数
>25 ms 帧数
>33.3 ms 帧数
>50 ms 帧数
最大 Frame Time
```

重点观察 P95、P99 和 1% Low。

## 18.2 World Partition Hitch

在 Insights 中对齐：

```text
Cell Load
Asset Load
Actor Initialize
Component Register
PCG Generate
Texture Streaming
Nanite Streaming
```

如果 Cell 加载造成峰值：

- 调整 Cell Size；
- 调整 Loading Range；
- 提前添加 Streaming Source；
- 降低单 Cell Actor/组件数量；
- 使用 HLOD；
- 把 PCG 生成分散到多帧；
- 减少加载时复杂碰撞和导航更新。

## 18.3 PCG Hitch

检查：

- 是否一个帧同时调度过多 Cell；
- `NumGeneratingComponents` 是否过高；
- PCG Frame Time Budget；
- 是否反复生成/清理同一范围；
- Cleanup Radius 是否太接近 Generation Radius；
- 是否存在对象池；
- 是否在生成时同步加载 Mesh/Material。

## 18.4 Shader 与 PSO

如果第一次出现材质或效果时卡顿，而第二次不再发生，检查：

- Shader 是否预编译；
- PSO Precache；
- 打包版本是否包含所需 Shader；
- 是否有运行时创建的新材质排列；
- 是否在进入镜头时首次加载大资源。

作品集可以展示一次“冷运行”和一次“热运行”帧时间曲线。

## 18.5 阶段验收

- 60 秒路线 P95/P99 达到目标；
- >33.3 ms Hitch 数明显下降；
- 冷启动峰值有解释；
- 热运行不存在重复加载抖动；
- Streaming 和 PCG 调度能够跟上摄像机速度。

---

# 19. 建立画质等级与设备配置

真正的项目不会只有一台电脑和一个画质档。

## 19.1 建议画质档

```text
High
├── Lumen GI
├── Lumen Reflections
├── 完整 VSM
├── 较高植被密度
└── 完整环境特效

Medium
├── Lumen GI 或较低 Lumen 质量
├── SSR 替代部分 Lumen Reflections
├── 降低植被阴影和密度
├── 降低体积雾
└── 更积极 HLOD/Cull

Low
├── 简化 GI/Skylight
├── SSAO
├── 更低植被密度
├── 关闭非关键流体
└── 简化远景材质
```

## 19.2 不要留下“控制台命令汤”

调试完成后，把确认有效的设置整理到：

```text
Scalability Settings
Device Profiles
DefaultScalability.ini
DefaultEngine.ini
资产自身设置
PCG Graph 参数
Material Quality Switch
```

作品集列出正式保留的设置，而不是贴几十条临时 CVar。

## 19.3 每个画质档都测试

记录：

| 档位 | 输出分辨率 | 内部分辨率 | 平均 FPS | 1% Low | GPU P95 | VRAM |
|---|---:|---:|---:|---:|---:|---:|
| High | | | | | | |
| Medium | | | | | | |
| Low | | | | | | |

---

# 20. 最终回归测试

## 20.1 最终对照条件

必须确保：

```text
相同硬件
相同驱动
相同引擎版本
相同打包配置
相同地图路线
相同分辨率
相同 Screen Percentage
相同 Lumen 模式
相同 VSM 模式
相同画质档
相同测试时长
```

## 20.2 三类最终测试

### A. 热运行性能

资源已经加载和预热，测稳定帧率。

### B. 冷启动性能

重启程序后第一次运行，测：

- 首次 Shader/PSO；
- 纹理加载；
- Nanite Streaming；
- World Partition；
- PCG；
- IO。

### C. 压力测试

加入：

- 高速飞行；
- 快速转身；
- 镜头硬切；
- 传送；
- 从低处快速升到高空；
- 在森林最密集区域停留。

## 20.3 最终数据表

```markdown
| 指标 | Baseline | Optimized | 变化率 |
|---|---:|---:|---:|
| 平均 FPS | | | |
| 1% Low FPS | | | |
| P95 Frame Time | | | |
| P99 Frame Time | | | |
| Game Thread P95 | | | |
| Render Thread P95 | | | |
| GPU P95 | | | |
| Draw Calls | | | |
| Actor 数 | | | |
| 组件数 | | | |
| VRAM 峰值 | | | |
| RAM 峰值 | | | |
| Texture Pool | | | |
| >33.3 ms Hitch | | | |
| 打包体积 | | | |
```

变化率：

```text
性能数值降低类：
(Baseline - Optimized) / Baseline × 100%

FPS 提升类：
(Optimized - Baseline) / Baseline × 100%
```

## 20.4 视觉回归

至少检查：

- 白天顺光；
- 逆光；
- 森林内部；
- 远景悬崖；
- 水面和反射；
- 快速运动；
- HLOD 切换；
- 树叶风动画；
- 阴影过渡；
- Lumen GI 和反射；
- 近景纹理清晰度。

---

# 21. 作品集页面结构

## 21.1 第一页：结果先行

包含：

```text
项目标题
引擎版本
目标硬件
目标分辨率
优化前后 FPS
优化前后 P95 Frame Time
优化前后 VRAM/RAM
一张 Before/After 主图
```

不要在第一屏先写长篇背景。

## 21.2 第二页：问题和预算

说明：

- Electric Dreams 是 4×4 km 高规格环境；
- 完整地图默认 Streaming Disabled；
- 使用 PCG、Nanite、Lumen、VSM、Substrate 等；
- 你的目标硬件明显低于官方推荐或具有明确帧预算；
- 你不允许简单关闭核心视觉特性。

## 21.3 第三页：测试方法

展示：

- Benchmark Sequence 路线图；
- 固定测试参数；
- 三次运行取中位数；
- Unreal Insights、ProfileGPU、CSV；
- 热运行和冷运行分开。

## 21.4 第四页：Baseline

展示：

- `stat unit`；
- GPU Pass 排名；
- 帧时间曲线；
- 内存；
- 3～5 个主要瓶颈；
- 不要一开始就放几十个问题。

## 21.5 重点案例一：World Partition + HLOD

采用结构：

```text
现象
→ 整个地图驻留，内存高。

证据
→ Streaming Disabled、常驻 Actor、内存曲线。

根因
→ 4×4 km 地图默认没有运行时 Cell 流送。

修改
→ Streaming、Runtime Grid、Streaming Source、HLOD。

结果
→ 填写 RAM、VRAM、Actor、Hitch 实际变化。

代价
→ 增加 Cell 调度、HLOD 制作和 Pop-in 控制工作。
```

## 21.6 重点案例二：PCG

展示：

- Graph Before/After；
- 节点点数；
- 节点 Profile；
- 提前过滤；
- Copy Points 属性精简；
- Runtime 与 Baked 分类；
- ISM/HISM；
- 生成 Hitch 曲线。

## 21.7 重点案例三：植被、Nanite、VSM

展示：

- Quad Overdraw；
- Nanite Overdraw；
- Evaluate WPO；
- VSM Cache；
- WPO 距离关闭；
- 小植被阴影策略；
- GPU Pass 前后对比。

## 21.8 重点案例四：材质/Lumen

展示：

- Shader Complexity；
- Material Analyzer；
- 材质距离分层；
- Surface Cache；
- Lumen Pass；
- 最终视觉对比。

## 21.9 最终总结页

包含：

- Baseline/Optimized 数据表；
- Frame Time 曲线；
- Before/After；
- 三项最大收益；
- 三项最大代价；
- 尚未解决的问题；
- 如果进入 UE6 后准备验证哪些内容。

---

# 22. 推荐视频脚本

总时长建议：2～4 分钟。

## 0:00～0:15 结果

```text
同一镜头 Before / After 分屏
显示目标硬件、分辨率和帧时间
```

## 0:15～0:35 项目与问题

```text
4×4 km Electric Dreams
默认关闭 Streaming
PCG、Nanite、Lumen、VSM
目标：60 FPS / 16.67 ms
```

## 0:35～1:10 World Partition/HLOD

```text
显示 Loaded Cell
显示内存下降
显示远处 HLOD
显示高速飞行无明显空洞
```

## 1:10～1:45 PCG

```text
显示 Graph 前后
显示点数变化
显示 ISM/HISM
显示生成 Hitch 变化
```

## 1:45～2:25 GPU

```text
Nanite Overdraw
植被 Quad Overdraw
WPO/VSM Cache
材质复杂度
Lumen Pass
```

## 2:25～3:00 最终数据

```text
Frame Time 曲线
1% Low
GPU P95
RAM/VRAM
视觉对比
```

录制建议：

- 录屏和正式测试分开；
- 不要把录屏时 FPS 当最终数据；
- Before/After 使用相同摄像机、相同时间、相同曝光；
- 图表上写毫秒，不要只写 FPS；
- 明确标注任何画质变化。

---

# 23. 四周执行计划

## 第 1 周：Baseline 与 World Partition

### 第 1 天

- 安装项目；
- 确认引擎版本；
- 复制地图；
- 建立版本管理；
- 填写硬件和目标。

### 第 2 天

- 制作 Benchmark Sequence；
- 固定分辨率和画质；
- 打包 Development/Test；
- 验证路线可重复。

### 第 3 天

- 采集 Baseline；
- Unreal Insights；
- ProfileGPU；
- CSV；
- 内存和纹理数据。

### 第 4～5 天

- 开启 World Partition Streaming；
- 调整 Runtime Grid；
- 检查 Spatially Loaded；
- 配置 Streaming Source；
- 测试快速飞行。

### 第 6～7 天

- 建立 HLOD；
- 检查远景和切换；
- 完成第一阶段报告。

## 第 2 周：PCG 与 CPU/Render Thread

- 分类 Baked 和 Runtime PCG；
- PCG Profiling；
- 提前过滤点；
- Copy Points 属性精简；
- ISM/HISM；
- 关闭装饰碰撞、导航和 Tick；
- 测试加载 Hitch。

## 第 3 周：GPU

- Nanite Mask/Overdraw；
- 非 Nanite LOD；
- 植被 Quad Overdraw；
- WPO 距离；
- VSM Cache；
- Material Analyzer；
- Lumen Surface Cache；
- ProfileGPU 回归。

## 第 4 周：内存、回归与作品集

- Size Map 和 Asset Audit；
- 纹理预算；
- Memory Insights；
- 冷启动/热运行；
- High/Medium/Low；
- 最终三次测试；
- Before/After 录制；
- 图表和案例文章；
- 视频剪辑。

---

# 24. 命令与视图速查表

## 24.1 基础性能

```text
stat unit
stat unitgraph
stat fps
stat gpu
stat rhi
stat scenerendering
profilegpu
```

## 24.2 纹理与内存

```text
stat streaming
stat memory
r.Streaming.PoolSize
```

完整 Memory Trace 启动参数：

```text
-trace=default,memory
```

## 24.3 固定测试环境

```text
r.VSync 0
t.MaxFPS 0
r.DynamicRes.OperationMode 0
r.ScreenPercentage 100
```

## 24.4 CSV

```text
csvprofile start
csvprofile stop
csvprofile frames=3600
```

## 24.5 Lumen

```text
r.Lumen.Visualize.CardPlacement 1
r.Lumen.Visualize.CardPlacement 0
```

## 24.6 PCG Runtime

```text
pcg.RuntimeGeneration.Enable
pcg.RuntimeGeneration.EnableDebugging
pcg.RuntimeGeneration.EnablePooling
pcg.RuntimeGeneration.NumGeneratingComponents
pcg.FrameTime
```

## 24.7 重要 View Modes

```text
Shader Complexity
Quad Overdraw
Light Complexity
Required Texture Resolution
Material Texture Scale Accuracy
Nanite Overview
Nanite Mask
Nanite Overdraw
Nanite Evaluate WPO
Lumen Overview
Lumen Scene
Lumen Surface Cache
Lumen Performance Overview
Virtual Shadow Map Visualization
World Partition Cell/Grid
```

---

# 25. 最终检查清单

## 项目与测试

- [ ] 写明引擎版本；
- [ ] 写明目标硬件；
- [ ] 写明分辨率和 Screen Percentage；
- [ ] 使用固定 Sequence；
- [ ] Baseline 和 Optimized 条件相同；
- [ ] 使用打包版本；
- [ ] 每个版本至少测试三次；
- [ ] 使用中位数；
- [ ] 冷启动和热运行分开；
- [ ] 不使用录屏数据作为最终结果。

## World Partition/HLOD

- [ ] Streaming 已开启；
- [ ] Runtime Grid 有测试依据；
- [ ] Spatially Loaded 配置正确；
- [ ] Streaming Source 能跟上 Drone；
- [ ] HLOD 已生成；
- [ ] 远景没有空洞；
- [ ] Cell 切换没有严重 Hitch；
- [ ] 内存能随卸载回落。

## PCG

- [ ] 每个 Graph 已分类 Baked/Runtime；
- [ ] 使用 PCG Profiling；
- [ ] 复杂操作前已过滤点；
- [ ] Copy Points Attribute 已检查；
- [ ] 重复 Mesh 使用实例；
- [ ] 装饰物关闭不必要碰撞；
- [ ] Runtime Generation 有 Hierarchical 策略；
- [ ] 没有频繁生成/清理抖动。

## GPU

- [ ] ProfileGPU 前后对比；
- [ ] Nanite Overdraw 已检查；
- [ ] 植被 Quad Overdraw 已检查；
- [ ] WPO 距离已检查；
- [ ] VSM Cache 已检查；
- [ ] Lumen Surface Cache 已检查；
- [ ] 材质 Complexity 已检查；
- [ ] 透明、雾和后处理有实际证据。

## 内存

- [ ] Size Map；
- [ ] Asset Audit；
- [ ] Memory Insights；
- [ ] Texture Pool 不持续超预算；
- [ ] VRAM 低于目标；
- [ ] 没有无意义 8K；
- [ ] 没有错误 Never Stream；
- [ ] Nanite Streaming 不持续抖动。

## 作品集

- [ ] 第一页直接展示结果；
- [ ] 有测试方法；
- [ ] 有 Baseline；
- [ ] 每项优化有证据、修改、收益和代价；
- [ ] 有毫秒数据；
- [ ] 有 1% Low 或 P95/P99；
- [ ] 有视觉 Before/After；
- [ ] 有失败尝试或取舍说明；
- [ ] 没有把降分辨率伪装成内容优化；
- [ ] 没有公开分发不应公开的完整官方样例资源。

---

# 26. 官方参考资料

- [Electric Dreams Environment](https://dev.epicgames.com/documentation/en-us/unreal-engine/electric-dreams-environment-in-unreal-engine)
- [Procedural Content Generation in Electric Dreams](https://dev.epicgames.com/documentation/en-us/unreal-engine/procedural-content-generation-in-electric-dreams)
- [Performance Profiling and Configuration](https://dev.epicgames.com/documentation/en-us/unreal-engine/introduction-to-performance-profiling-and-configuration-in-unreal-engine)
- [World Partition](https://dev.epicgames.com/documentation/unreal-engine/world-partition-in-unreal-engine?lang=en-US)
- [World Partition HLOD](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine)
- [PCG Runtime Generation](https://dev.epicgames.com/documentation/en-us/unreal-engine/using-pcg-generation-modes-in-unreal-engine)
- [PCG Runtime Generation Debugging](https://dev.epicgames.com/documentation/unreal-engine/pcg-runtime-generation-debugging-in-unreal-engine)
- [Nanite Technical Details](https://dev.epicgames.com/documentation/en-us/unreal-engine/nanite-technical-details)
- [Lumen Technical Details](https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-technical-details-in-unreal-engine)
- [Lumen Performance Guide](https://dev.epicgames.com/documentation/en-us/unreal-engine/lumen-performance-guide-for-unreal-engine)
- [Virtual Shadow Maps](https://dev.epicgames.com/documentation/en-us/unreal-engine/virtual-shadow-maps-in-unreal-engine)
- [Texture Streaming](https://dev.epicgames.com/documentation/unreal-engine/texture-streaming-in-unreal-engine?lang=en-US)
- [Memory Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/memory-insights-in-unreal-engine)
- [Material Analyzer](https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-material-analyzer-tool?lang=en-US)

---

## 建议首先完成的三个 Sprint

如果不想一开始同时处理所有内容，严格按以下顺序执行：

### Sprint 1：建立证据

```text
固定路线
→ 打包版本
→ Baseline
→ Unreal Insights
→ ProfileGPU
→ 数据表
```

### Sprint 2：处理场景规模

```text
World Partition Streaming
→ Runtime Grid
→ Streaming Source
→ HLOD
→ 内存和 Hitch 回归
```

### Sprint 3：处理最重镜头

```text
找到 GPU 最重镜头
→ Nanite/Quad Overdraw
→ 植被 WPO
→ VSM Cache
→ 材质/Lumen
→ 前后 ProfileGPU
```

只有前三个 Sprint 都有完整前后数据后，再扩展到纹理、流体、Niagara 和多画质档。
