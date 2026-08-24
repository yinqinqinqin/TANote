# Random Grain

## Blender中的乱纹不锈钢效果

![image-20260824150843412](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824150843645.png)

- **UV Mapping**
  - 调整 UV 缩放，让划痕纹理重复分布。

- **采样方向纹理**
  - 使用 `scratches_vector.tif` 保存每个像素的划痕方向。

- **方向数据重映射**
  - 将纹理值减去 `0.5`，把 `0~1` 转换为以 `0` 为中心的方向数据。

- **分离 XY**
  - 获取二维方向向量的 `X`、`Y` 分量。

- **Atan2 计算角度**
  - 使用 `atan2(Y, X)` 将二维方向向量转换为旋转角度。

- **角度调整**
  - 对角度进行缩放和偏移，使其符合 `Anisotropic Rotation` 的输入范围。

- **控制各向异性方向**
  - 将最终结果连接到 `Anisotropic Rotation`。

- **最终效果**
  - 每个像素根据划痕方向旋转各向异性高光，实现随机拉丝 / 乱纹不锈钢效果。



![image-20260824154713470](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824154713662.png)

## UE中乱纹效果

![image-20260824153555551](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824153555698.png)

![image-20260824151603137](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824151603363.png)

![image-20260824152859744](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824152859941.png)

- 采样划痕方向纹理

- 减去 `0.5`，得到正负方向数据

- 输入 `Tangent`，控制各向异性方向

- 放大后输入 `Anisotropy`，增强高光拉伸

- 使用纹理 Alpha 控制低 `Roughness`

- 最终得到随机方向的细碎各向异性高光



## 贴图分析

R

![image-20260824154247504](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824154247622.png)

G

![image-20260824154334943](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824154335050.png)

A

![image-20260824154432250](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824154432370.png)

R，G控制滑痕方向

A控制粗糙度