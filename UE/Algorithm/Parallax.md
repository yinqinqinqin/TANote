# Parallax

```glsl
// 自定义节点输出类型：Float2（严格匹配你的设置）
// 输入：uv(float2), viewDir(float3), Tangent(float3), VertexNormalWS(float3), offset(float)

float3 worldRayDir = -viewDir;
float3 Bitangent = -cross(Tangent, VertexNormalWS);

// 标准切线空间视线转换
float3 tsViewDir;
tsViewDir.x = dot(worldRayDir, Tangent);
tsViewDir.y = dot(worldRayDir, Bitangent);
tsViewDir.z = dot(worldRayDir, VertexNormalWS);

tsViewDir = normalize(tsViewDir);
float2 uvOffset = tsViewDir.xy / max(tsViewDir.z, 0.001f) * offset;
float2 finalUV = uv - uvOffset;

// 所有超出范围的UV都会被限制在纹理边缘，不会返回原始UV
finalUV = clamp(finalUV, 0.0f, 1.0f);

return finalUV;
```

