# ElectricDreams优化

## 目标

### 硬件信息

```
操作系统：Windows 11 专业版 64 位（Build 26200）
CPU：Intel Core Ultra 7 265K（20 核 / 20 线程）
GPU：NVIDIA GeForce RTX 5060
显存：8 GB（系统检测约 8151 MiB）
系统内存：64 GB（系统可用约 63.7 GiB）
硬盘：WD_BLACK SN7100 2TB NVMe SSD
显示分辨率：2560 × 1440，约 144 Hz
目标输出分辨率：2560 × 1440（建议使用显示器原生分辨率）
内部渲染比例：100%（建议作为性能测试基线）
目标帧率：60 FPS
目标帧时间：16.67 ms
```

### 优化目标

```
测试案例：Electric Dreams Environment
测试地图：/Game/Levels/ElectricDreams_Env
测试构建：Windows Development 打包版本
渲染接口：DirectX 12
目标输出分辨率：1920 × 1080
内部渲染比例：50%
目标帧率：60 FPS
目标帧时间：16.67 ms
最低可接受帧率：30 FPS
最低可接受帧时间：33.33 ms
```

### 影响帧率因素

- Lumen GI：开启；
- Lumen Reflections：开启或使用 SSR，必须写明；
- Hardware Ray Tracing：开启或关闭，必须固定；
- Virtual Shadow Maps：开启；
- Nanite：开启；
- TSR：/开启；
- 输出分辨率固定；
- `r.ScreenPercentage` 固定；
- Scalability 固定。

## 版本管理

## 固定运行条件

在正式采集前输入：

```text
r.VSync 0
t.MaxFPS 0
r.DynamicRes.OperationMode 0
```

## 采集数据

### 记录线程

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

**记录**

```text
Frame：总帧时间Z
Game：Game Thread
Draw：Render Thread
GPU：GPU 帧时间
```

### 记录GPU Pass

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

### 记录渲染规模

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

### 记录纹理流送

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

## 判断 CPU、GPU 或内存瓶颈

###  GPU Bound

特征：

- `stat unit` 中 GPU 明显最高；
- 降低分辨率后 FPS 明显提升；
- ProfileGPU 中某个 Pass 耗时突出。

###  Game Thread Bound

特征：

- Game Thread 最高；
- 降低分辨率没有明显变化；
- Insights 中大量 Tick、PCG、碰撞、蓝图或对象生成。

###  Render Thread Bound

特征：

- Draw/Render Thread 最高；
- Draw Call 和 Primitive 数量大；
- 大量独立 Actor 和独立材质状态。

### 内存或流送问题

特征：

- 平均帧正常，但移动时卡顿；
- `stat streaming` 经常 Over Budget；
- VRAM 接近物理上限；
- Nanite 或纹理反复 Stream In/Out；
- World Partition Cell 加载时出现几十毫秒停顿。

## 优化方法
