# 天空球体

当我们需要在天空添加星星，月亮时，需要一个天空球体的材质，在材质中添加相关的物体

1. 使用引擎内置的球形网格体，该网格体的法线都是朝内的，确保光线可以穿透球体
   1. 确保设置中启用了现实引擎内容和插件内容
   2. 搜索sky，找到SM_SkySphere
   3. 给网格体添加自定义材质，并将缩放调整到一个极大值
2. 材质制作
   1. 基础设置
      1. 将着色模型从Lit改为Unlit
      2. 勾选是否为天空材质
      3. 需要外部数据模拟天空球行为
   2. 材质
      1. 需要SkyAtmosphereviewLuminance（视场亮度）：包含天空球的颜色信息
      2. SkyAtmosphereLightDiskLuminance：场景中的定向光源
      3. 添加之后的区别![image-20260713152128293](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260713153917056.png)