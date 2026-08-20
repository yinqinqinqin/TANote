# 爱弥斯角色渲染

## 1. 项目概述

- 项目：爱弥斯角色渲染复刻
- 引擎：UE5.7
- 方向：NPR 角色渲染

------

# 2. 模型与材质结构

## 模型

- Hair
- HairBack
- Eyes
- Face
- HeartFX
- Item
- Cloth-Up
- Cloth-Down

## 材质

- Hair
- Eyes
- Face
- Item
- HeartFX
- Cloth

------

# 3. 基础材质制作步骤

## 3.1 获取贴图通道和材质ID

### BaseColor

- RGB：BaseColor
- A：Shadow / Emissive

### Normal

- RG：Normal
- A：Roughness

### FTM

- R：Opacity
- G：Metallic
- B：战损 Mask

### ID

用于区分不同材质区域。

------

## 3.2 颜色调整

使用：

- MX_HSVAdjust
- MF_Tonemap

```hlsl
#include "/Engine/Private/TonemapCommon.ush"
return FilmToneMapInverse(Input);
```

------

## 3.3 光照

### Lambert

```hlsl
float NdotL = dot(N, L);
```

用于获得基础明暗关系。

### MatCap 风格化高光

将法线转换到 View Space：

```hlsl
float2 MatCapUV = NormalVS.xy * 0.5 + 0.5;
```

流程：

```text
NormalWS
↓
NormalVS
↓
取 XY
↓
0~1
↓
采样 MatCap
```

------

## 4. 阴影

## 4.1三色过渡明暗分层

普通 Toon 通常是：

```text
暗部 → 亮部
```

这里增加一个中间色：

```text
暗部 → 中间色 → 亮部
```

使用两个 Remap 区间产生两个 Mask：

```hlsl
float mask1 = saturate(
    (L - DarkMin) /
    (DarkMax - DarkMin)
);

float mask2 = saturate(
    (L - LightMin) /
    (LightMax - LightMin)
);

float3 color =
    lerp(DarkColor, MidColor, mask1);

color =
    lerp(color, LightColor, mask2);
```

核心：

```text
一个阈值 → 两个颜色
两个阈值 → 三个颜色
```

------

## 5.透明度

FTM 的 R 通道作为 Opacity。

用于控制衣服部分的透明区域。

------

## 6. 最终结构

```text
贴图
↓
颜色调整
↓
Lambert
↓
三色光照
↓
MatCap
↓
Voronoi 闪粉
↓
透明度
↓
最终材质
```

# 特殊材质制作

## 闪粉效果

使用程序化 Voronoi 生成随机法线。

流程：

```text
Position
↓
Voronoi
↓
随机 Cell
↓
随机 Normal
↓
MatCap
↓
闪粉
```

将随机值从：

```text
0~1
```

转换到：

```text
-1~1
```

作为法线方向：

```hlsl
float3 n = finalColor * 2.0 - 1.0;

n.z = abs(n.z) + 0.2;

n = normalize(n);

return n * 0.5 + 0.5;
```

再与原始 Normal 混合，通过 MatCap 产生闪粉高光。

# 视差效果

基于切线空间 View Direction 的简单视差偏移

原理

```glsl
  // 输入：uv, viewDirTS（切线空间）, strength float3 V = normalize(viewDirTS); 
  // 只用方向，不用深度 float2 offset =V.xy * strength;
   // 输出 return uv + offset;
    
    uv = uv *2.0-1.0; 
    uv = uv * ParallaxMap_ST.xy + ParallaxMap_ST.zw; 
    uv = uv *0.5+0.5; 
    float cosTheta = dot (viewTS,float3(0,0,1)); 
    float ViewTSLength = depth / cosTheta; 
    float3 startPoint = float3(uv,0); 
    float3 endPoint = startPoint + viewTS * ViewTSLength;
     return endPoint.xy;
```

# 文字流动效果





# 麦克风消散效果



# 场景制作

