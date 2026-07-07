# RayMarching

UE中的光线步进

输入：camera Position,Absolute World Position , Object Position , Radius , Color

custom节点

```glsl
#define MIN_STEP 1
#define MAX_MARCHINGTIMES 2560

float3 RayOrigin = CameraPos;
float3 RayStep = normalize(WorldPos - CameraPos) * MIN_STEP;

for(half i = 0; i < MAX_MARCHINGTIMES; i++)
{
    RayOrigin += RayStep;
    float Dist = length(RayOrigin - ObjectPos);
    if(Dist < SphereRadius)
    {
        SphereMask = 1;
        return SphereColor;
    }
}
SphereMask = 0;

return 0;
```

