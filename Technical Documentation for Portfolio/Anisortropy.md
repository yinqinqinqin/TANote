# Stalized Anisortropy

## 风格化各向异性原理

- 使用代码生成各向异性高光形状
- 确定好各向异性高光位置
- 根据视线与物体之间的夹角进行计算，以近似真实各项异性效果

各项异性基本代码

 ````
 float3 N_TS = TransformWorldToTangent(NormalWS);
 
 float3 A_TS =
     cross(float3(1.0, 0.0, 0.0), N_TS)
     + N_TS * HeightTex;
 
 float3 A_WS = TransformTangentToWorld(A_TS);
 
 float AV = dot(A_WS, CameraDir);
 
 float AnisoMask =
     sqrt(saturate(1.0 - AV * AV));
 
 AnisoMask =
     saturate(pow(AnisoMask, AnisoPower));
 
 
 float NoV = dot(NormalWS, CameraDir);
 
 float ViewMask =
     saturate(pow(NoV, RimMaskPower));
 
 
 float3 FinalColor =
     AnisoMask
     * ViewMask
     * HighlightColor;
 ````

## 风格化头发

![image-20260824170148081](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824170148193.png)
