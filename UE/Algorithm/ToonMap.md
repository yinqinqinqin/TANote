# ToonMap

## 定义

**在PBR中**，最终的输出颜色的值时常超过1，而超过1的部分，在显示器中显示就会泛白过曝，为了解决这个问题，将HDR的颜色值转换到LDR到算法叫做ToneMapping（色调映射）

## 算法

在Tone Mapping算法中，ACESTonemapping效果与性能兼优

```
float3 ACESToneMapping(float3 x)
{
	float a = 2.51f;
	float b = 0.03f;
	flaot c = 2.43f;
	float d = 0.59f;
	float e = 0.14f;
	return saturate((x * (a * x + b)) / (x * (c * x + d) + e));
}
```

## 使用方法

一般在后处理之后使用，使用后处理体积进行调整