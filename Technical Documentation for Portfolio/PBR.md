# PBR

眼睛看到的物体颜色是光线照射到物体上，反射出来，而PBR表示的是光线反射的真实情况

如果需要自行实现 PBR 光照，可以使用 Unlit 绕开 UE 默认着色模型，并通过 Emissive 输出自行计算完成的光照结果。

在真实渲染中，物体会有四种反射情况

|        | 漫反射                                                       | 镜面反射                                                     |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 直接光 | LightColor * Kd * BaseColor / pai  * N dot L （Lambert）     | LightColor * D * F * G / (4 * NdotL * NdotV) * NdotL;（Cook-Torrance） |
| 间接光 | IBL → Irradiance（可用 SH / Irradiance Map 表示） / Lumen GI | 环境光反射 + LUT   Lumen Reflections                         |

Kd 表示进入漫反射部分的能量比例

F0 表示 lerp(Spacular(0.04 ) , Basecolor ,Metallic)

F0 = leap(0.04,Basecolor,Metallic)

ks = F0;

kd = (1.0 - ks) * (1.0 - Metallic);

直接光条件：

- 微表面
- 能量守恒
- BRDF



# BRDF公式

参考BRDF公式对光照进行计算

![image-20260708111805170](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260820160909586.png)



简化公式

![image-20260825153700815](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260825153700980.png)

## D

法线分布函数 GGX

![image-20260820164611147](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260820164611444.png)

alpha表示 Roughness * Roughness

H表示 normalize(V + L)半角向量

```
float alpha  = Roughness * Roughness;
float alpha2 = alpha * alpha;

float NdotH = saturate(dot(N, H));

float temp = NdotH * NdotH * (alpha2 - 1.0) + 1.0;

float D =
    alpha2 /
    (PI * temp * temp);
```

## F

决定当前观察角度下，有多少光发生镜面反射。

![image-20260820164725592](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260820164725736.png)



```
float VdotH = saturate(dot(V, H));

float3 F =
    F0 +
    (1.0 - F0)
    * pow(1.0 - VdotH, 5.0);
```

### G

微表面之间因为互相遮挡，导致多少光真正可以参与反射。

![image-20260820165150552](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260820165150845.png)

X 表示 V或L其中一种

k 是根据 Roughness 推导出来的 Schlick-GGX 几何遮挡近似系数，用于控制微表面的 Masking 和 Shadowing。

```
// Direct Lighting
float k = pow(Roughness + 1.0, 2.0) / 8.0;

// IBL
float k = Roughness * Roughness / 2.0;

float Gv =
    NdotV /
    (NdotV * (1.0 - k) + k);

float Gl =
    NdotL /
    (NdotL * (1.0 - k) + k);

float G = Gv * Gl;

```

## 自定义单光源PBR

### 1.设置基础属性

1. 设置颜色 BaseColor
2. 金属度 Metallic
3. 粗糙度 Roughness
4. 法线 Normal
5. 视线方向 ViewDir
6. 光源方向 LightDir
7. H  = normalize(V + L);

### 2.计算基础数据

1. NOH = Dot(N , H);
2. NOV = Dot(N , V);
3. NOL = Dot(N , L);
4. F0 = Leap(0.04(Spacular) , BaseColor , Metallic);
5. ks = 

### 3.计算D，F，G

D：

![image-20260820164611147](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824171347464.png)

![image-20260824171642462](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824171642582.png)

F：![image-20260820164725592](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824171415128.png)

![image-20260824171752328](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824171752452.png)

G： 

![image-20260820165150552](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824171809484.png)

![image-20260708141610146](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260825112037651.png)

![image-20260825111842410](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260825111842527.png)

## 4.计算直接光

直接光漫反射

![image-20260825112943582](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260825112943745.png)

直接光镜面反射
