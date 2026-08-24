# VAT

![image-20260824162051090](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824162051358.png)

在Blender中制作动画效果，将动画效果通过插件烘培到贴图上

UE通过当前帧+顶点ID采样纹理，得到顶点位移

```
Position Texture
→ 每个顶点每一帧的位置 / 位移

Normal Texture
→ 每个顶点每一帧的法线


             Frame
          0   1   2   3   4
Vertex 0  ●   ●   ●   ●   ●
Vertex 1  ●   ●   ●   ●   ●
Vertex 2  ●   ●   ●   ●   ●
Vertex 3  ●   ●   ●   ●   ●
```

u值代表帧数

v值代表顶点ID

![image-20260824162512330](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824162512506.png)

在UE中通过vertexAnimationTools这个节点，对顶点动画贴图进行解析

![image-20260824162604797](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824162604932.png)