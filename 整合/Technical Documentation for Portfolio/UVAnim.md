# UVAnim

![image-20260824160511469](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824160511550.png)

通过修改uv的路径对动画进行修改

![image-20260824160632758](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824160632960.png)

第一个uv表示的是整个uv运动的路径

![image-20260824160718366](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824160718525.png)

其余的UV表示对图形的采样

![image-20260824160755399](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824160755496.png)

通过对模型的处理我们可以用更便宜的方式实现物体的运动

**注意**：采样时需要注意UV的拉伸

UE中处理拉伸问题

![image-20260824161103896](https://yin-qin.oss-accelerate.aliyuncs.com/img/20260824161104043.png)

- `TexCoord` 拆出 `U、V`。
-  `U - Time`：让 UV 随时间横向移动。
-  `+ 0.5 → Frac → - 0.5`：把移动后的 U 做成 **循环的、以 0 为中心的区间**。
-  `× -1`：反转 U 方向。
-  `/ 0.123`：把这一小段 UV 放大，相当于控制效果的**横向宽度**。
-  `+ 0.5`：重新把中心移回 `0.5`。
-  原来的 `V` 不变。
-  最后 `Append(U,V)` 得到新的 UV。