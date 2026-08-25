# VolumeCloud

## UE 体积云系统

![image-20260825164401074](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260825164401214.png)

![image-20260825162343793](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260825162344090.png)

这部分先使用世界空间 2D Mask 控制云的大范围分布，再叠加世界空间 3D Noise 对云进行侵蚀和细节塑形，最终生成用于体积云的 Extinction/Density。

## UE光线步进

输入：

- CameraPos
- WorldPos
- ObjectPos
- SphereRadius

```
#define MIN_STEP 1
#define MAX_MARCHINGTIMES 1024

float3 RayOrgin = CameraPos;
float3 RayStep = normalize(WorldPos - CameraPos) * MIN_STEP;

for(half i = 0; i < MAX_MARCHINGTIMES; i++)
{
    RayOrgin += RayStep;
    float Dist = length(RayOrgin - ObjectPos);
    if(Dist < SphereRadius)
    {
        return 1;
    }
}

return 0;
```

流程：

```
CameraPos
   ↓
计算 Camera → WorldPos 的射线方向
   ↓
每次前进 MIN_STEP
   ↓
计算当前采样点到球心 ObjectPos 的距离
   ↓
Dist < SphereRadius ?
   ├─ 是 → return 1
   └─ 否 → 继续步进
   ↓
超过最大步数
   ↓
return 0
```

## Unity手搓体积云

### sdf光线步进

### 3dTexture体积云

