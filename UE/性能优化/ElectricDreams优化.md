# ElectricDreams优化

## Nanite减面优化-石头

### 初始状态

![image-20260730104820624](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260730104820731.png)

### 优化步骤

1. 缩小三角形百分比
   1. 保持三角形百分比：构建 Nanite 数据时，保留原始模型多少比例的三角形。
   2. 回退三角形百分比：Nanite 之外生成的传统静态网格还保留多少三角形。
   3. ![image-20260730105850172](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260730105851035.png)
   4. ![image-20260730112935362](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260730112935580.png)
   5. 面数明显降低

## 窗口介绍

### 视口可延展性 

![image-20260730103055964](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260730103056249.png)

**视野距离**：物体多远开始降级或被裁剪。

**抗锯齿（TSR）**：TSR、TAA 等抗锯齿和重建质量。

**后期处理**：景深、Bloom、环境光遮蔽、色调映射等。

**阴影**：阴影分辨率、距离、级联数量等。

**全局光照**：主要影响 Lumen GI 等。

**反射**：Lumen Reflection、SSR 等反射质量。

**纹理**：纹理流送与 Mip 质量。

**特效**：粒子、体积效果等。

**植物**：植被密度与显示距离。

**着色**：部分材质和着色质量设置。

## 屏幕百分比

![image-20260730103526263](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260730103526373.png)

一般使用68

## 优化视图模式

![image-20260730103647775](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260730103647853.png)