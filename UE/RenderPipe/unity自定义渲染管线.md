# unity自定义渲染管线

![image-20260812142518231](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260812142518901.png)

## 渲染的方式

### 光线步进

![image-20260812145711468](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260812145711591.png)

### SDF光线步进

![image-20260812145626375](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260812145626728.png)

## 重要的RT

### Color RT

屏幕最终看到的颜色

### Depth RT

遮挡问题

### 渲染管线的顺序

真正的渲染管线是绘制物体的顺序

1. 画深度图
2. 画不透明物体
3. 画天空球
4. 画不透明物体
5. 后处理

## 前向渲染

![image-20260812163833831](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260812163834132.png)

定义多光源数组

## Early-Z

- GPU提供的技术
- 将深度测试放在像素着色器前面
- 只对实心物体有效
- 失效的情况
  - 手动修改深度值
  - 丢弃像素
  - 优化不稳定

## Z-Prepass

解决Early-Z失效问题

就是在真正画颜色之前，**先把不透明物体的深度画一遍**。

- 自定义
- 需要与Early-Z配合
- 分成两个pass实现
  - Pass1:仅写入深度，不写入颜色![image-20260812171048253](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260812171048608.png)
  - Pass2:关闭深度写入，ZTest Equal![image-20260812171330974](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260812171331152.png)

## Hi-Z



## 延迟渲染

### 什么是延迟渲染

==在后处理阶段进行光照处理==

需要的RT

- BaseColor
- Smothness
- Metallic
- Normal
- Depth

### 为什么要进行延迟渲染

解决复杂光照的问题

### 怎么做延迟渲染

使用MRT技术

一次渲染多个RT

![image-20260813105100851](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260813105101500.png)

### G-Buffer设计

![image-20260813105243613](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260813105243715.png)

![image-20260813110433673](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260813110433837.png)

## 光照优化

==与Forward+，defferred+==有关==

###  Clustered  Lighting

### Tiled Lighting

![image-20260813124831288](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260813124831597.png)

## 自定义渲染管线

![image-20260813124956624](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260813124956748.png)
