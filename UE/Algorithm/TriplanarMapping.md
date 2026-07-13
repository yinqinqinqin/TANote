# TriplanarMapping(三平面映射)

## 作用

根据模型表面的法线方向，自动在三个世界坐标平面（YZ/XZ/XY）之间混合投射纹理坐标，解决不规则模型、无 UV 模型的纹理拉伸问题，常见于岩石、地形、硬表面等无需手工展 UV 的材质。

## 实现原理

这套节点的核心逻辑，用**层级式两次插值**替代标准三平面的三通道加权混合，实现更简单、性能更低。

1. **第一次混合**
   - 输入 A：YZ 平面 UV（X 轴向投影）
   - 输入 B：XZ 平面 UV（Y 轴向投影）
   - 混合 Alpha：法线 Y 分量 → 高对比度处理 → Saturate 钳制
   - 效果：法线越朝向 Y 轴，越偏向 XZ 平面 UV；法线 Y 分量越小，越偏向 YZ 平面 UV。
2. **第二次混合**
   - 输入 A：第一次混合的结果
   - 输入 B：XY 平面 UV（Z 轴向投影）
   - 混合 Alpha：法线 Z 分量 → 高对比度处理 → Saturate 钳制
   - 效果：法线越朝向 Z 轴，越偏向 XY 平面 UV；否则沿用前一步的混合结果。

![image-20260713120933825](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260713120934125.png)

此处的CheapConstrast是

## UE材质

![image-20260710161830917](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260710161831322.png)

