# InterviewWeb

## 如何提升模型的UV利用率

- 合理排布UV岛
- 重复或对称的部分重叠UV
- 按重要程度非配面积

## 什么时候会用到2U

“2U”一般指**第二套 UV（UV2）**

- 烘培静态光照
  - UV岛不能重叠
  - 保持足够间距
- 同一个模型需要两种不同的贴图映射
- 存储特殊材质数据

## 什么是贴花（Decal）？

贴花是把一张带有 **颜色、法线、粗糙度等信息的纹理投射或覆盖到已有模型表面**，不用修改原模型，就能增加局部细节。

## 卡通渲染描边有哪些实现方案

1. 反向外壳描边 Inverted Hull
   1. 复制一层模型，沿顶点法线向外膨胀，渲染背面并使用黑色材质。
   2. 优点：轮廓稳定，角色描边效果好。
       缺点：硬边、法线断裂处容易炸开；远近描边粗细不一致；增加一次模型渲染。
2. 后处理深度描边
   1. 比较当前像素与周围像素的 Scene Depth：
   2. 优点：不需要修改模型，整个场景都能使用。
       缺点：同一平面内部结构检测不到；透明物体支持较差；远处容易变细或闪烁。
3. 后处理法线描边
   1. 比较当前像素和周围像素的世界法线：
   2. 优点：能产生内部结构线。
       缺点：法线贴图噪声可能产生杂乱描边，需要设置阈值。
4. Custom Depth / Stencil 描边
   1. 优点：可以精确控制哪些物体描边。
       缺点：主要得到外轮廓；透明材质和遮挡关系需要额外处理。
5. 遮罩膨胀描边
   1. 先把角色渲染成黑白 Mask，再对 Mask 做邻域扩张：
6. 手工描边模型

## 次世代场景模型贴图制作需要输出哪些数据？一般几张图？分别放在哪些通道

次世代场景模型贴图分为两类：**烘焙辅助数据**和**最终导入引擎的贴图**

### 最终导入 UE 的贴图

一般是 **3 张基础贴图**

1. BaseColor
2. Normal
3. ORM
   1. ORM = Occlusion + Roughness + Metallic + “ Height/Mask”

## PBR流程通常有两种，简述下两种的区别，现在工作流程最常见的是哪一种，为什么选择这种

1. Metallic / Roughness 金属度工作流
   1. Base Color
      Metallic
      Roughness
      Normal
      AO
2. Specular / Glossiness 高光工作流
   1. Diffuse
      Specular
      Glossiness
      Normal
      AO

现在实时游戏中最常见的是：

> **Metallic / Roughness 金属度工作流。**

### 1. 更符合标准化生产

美术只需要判断：

```
这个区域是金属还是非金属？
```

不需要手动猜测复杂的 Specular 数值，团队资产更容易保持统一。

### 2. 不容易做出错误材质

高光工作流可以随意设置 Specular 颜色，可能把塑料做成彩色反射，产生不符合物理规律的结果。

金属度工作流对参数限制更明确。

### 3. 更节省贴图通道

Metallic 和 Roughness 都是灰度图，可以和 AO 打包到一张 RGB 贴图中。

而 Specular 通常是 RGB 彩色贴图，需要更多纹理数据。

### 4. 更适合 UE 和 Substance 流程

UE、Substance Painter、Megascans 等实时游戏生产流程，对 Metallic/Roughness 的支持非常成熟，资产交换更加方便。

## 什么是IBL？实时渲染中IBL影响了光照的哪些部分？

IBL 是 **Image-Based Lighting，基于图像的光照**。

它使用 HDR 环境贴图记录周围各方向的颜色和亮度，让物体能够受到整个环境的照明和反射，而不是只依赖点光、方向光等传统光源。

## 什么是颜色空间？介绍你知道的颜色空间类型，并阐述它们的区别？

### 常见颜色空间

1. SRGB
   1. 最常见的显示颜色空间，用于：
      1. 普通显示器
      2. 网页和图片
      3. 游戏最终画面
      4. Base Color 贴图

1. Linear RGB
   1. 光照计算
   2. 颜色混合
   3. BRDF
   4. 后处理计算
   5. HDR 渲染

```
Base Color、Emissive颜色贴图
→ sRGB开启

Normal、Roughness、Metallic、AO、Mask、Height
→ sRGB关闭
```

## MatCap的使用原理和应用场景

原理

首先把模型法线转换到**观察空间 View Space**，取法线的 X、Y 分量映射成 UV：

```
float3 viewNormal = TransformWorldToView(NormalWS);

float2 matcapUV = viewNormal.xy * 0.5 + 0.5;

float3 color = MatCapTexture.Sample(Sampler, matcapUV);
```

## 对于树的渲染，常见的性能优化方案有哪些

### 模型

1. 合理减少叶片过绘制
2. 制作 LOD
3. 删除无效结构
4. 减少材质槽
5. 使用贴图图集
6.  简化碰撞
7. Nanite 的选择

### 渲染方面

1. 优化 Masked 叶片过绘制
2. 控制双面材质
3. 简化叶片材质
4. 优化风动 WPO
5. 优化阴影
6. 使用实例化
7. 使用裁剪和 HLOD
8. 控制纹理资源

## 性能影响的指标

| 指标           | 主要影响                       |
| -------------- | ------------------------------ |
| Draw Call 数量 | CPU、Render Thread、RHI Thread |
| 顶点数量       | GPU Vertex Shader、蒙皮、WPO   |
| 像素覆盖面积   | GPU Pixel Shader               |
| Shader复杂度   | GPU计算和纹理采样              |
| Overdraw       | GPU重复像素计算                |

## 实时渲染中次表面散射有哪些实现方案，并说明优劣

次表面散射（SSS）指光线进入半透明材质内部，经过多次散射后，从附近位置重新射出。

- 屏幕空间次表面散射
- 预积分次表面散射
- 厚度图加透射光

## 什么是渲染排序

决定不同 Render Pass 以及同一 Pass 内多个物体的绘制先后顺序。

通常先渲染不透明物体，再渲染半透明物体：

不透明物体：通常从近到远

#### 物体之间排序正确，但模型内部错误

UE 通常以一个 Primitive 或 Mesh 为单位排序，而不是对每个三角形都精确排序。

如果三角形绘制顺序不正确，可能出现：

- 背面覆盖正面；
- 内外层关系错误；
- 相机转动时透明效果跳变。

这叫 **Self-Sorting Problem**。

| 类型              | 常见绘制顺序     | 深度测试   | 深度写入         | 排序目的         |
| ----------------- | ---------------- | ---------- | ---------------- | ---------------- |
| Opaque            | 近到远           | 开启       | 开启             | 减少 Overdraw    |
| Masked            | 近到远           | 开启       | 开启             | 减少 Overdraw    |
| Alpha Translucent | 远到近           | 通常开启   | 通常关闭         | 保证颜色混合正确 |
| Additive          | 顺序影响较小     | 通常开启   | 通常关闭         | 发光叠加         |
| UI                | 根据层级/Z Order | 视系统而定 | 通常不用场景深度 | 保证界面层级     |

## UE灯光排序

```
场景准备与裁剪
→ 深度 / Nanite可见性
→ DBuffer贴花
→ Base Pass写入GBuffer
→ 准备VSM等阴影数据
→ 计算直接光照
→ Lumen间接光与天空光
→ 反射
→ 天空、大气、体积雾
→ 半透明前向光照
→ 后处理与TSR
→ 最终画面
```

## 一份模型文件会记录哪些顶点数据

模型的常见顶点数据包括顶点位置、法线、切线、多套 UV 和顶点颜色。骨骼模型还会保存骨骼索引和骨骼权重

## 半透明渲染有哪几种方案

传统混合
```
FinalColor =
    SrcColor * Alpha +
    DstColor * (1.0 - Alpha);
```

Masked／Alpha Test

```
clip(OpacityMask - Threshold);
```

预乘 Alpha

**柔边粒子**，UI，烟雾

```
PremultipliedColor = OriginalColor * Alpha;
```

 Additive 加法半透明

常用**发光**

```
FinalColor = SrcColor + DstColor;
```

