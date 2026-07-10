# VoronoiBoom

## 作用

去除纹理的重复感觉

## 实现原理

给纹理叠加一层**低频、大尺度的随机扰动**：

- 用沃罗诺伊图把整个表面分割成大量不规则的大细胞块，细胞尺寸远大于基础纹理的平铺单元；
- 给每个细胞块分配一个独有的随机 UV 偏移量，让每个区块内的纹理都发生错位；
- 细胞之间用渐变蒙版做软过渡，避免出现硬边断层。

## UE代码

![image-20260710163421669](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260710163421955.png)

![T_VoronoiNoise_Packed](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260710171536066.PNG)