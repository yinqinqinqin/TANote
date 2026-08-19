# MatCap

计算UV

float Viewdir = normalize(Camera - WorldPositon);

float a = normalize(Viewedir x (0,0,1));

Float b = a x Viewer;

float x = a · N;

Float y = N · b;

float2 uv = append(a , b);

uv = uv *0.5+0.5;

手动构造一个面向相机的局部坐标系

```
float3 V = normalize(CameraPos - WorldPos);
float3 R = normalize(cross(V, float3(0,0,1)));
float3 U = cross(R, V);

float2 matcapUV;
matcapUV.x = dot(N, R);
matcapUV.y = dot(N, U);

matcapUV = matcapUV * 0.5 + 0.5;
```



直接使用真正的 View Space 相机坐标系	

```
float2 MatCapUV = NormalVS.xy * 0.5 + 0.5;
```

