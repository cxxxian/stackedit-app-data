# 设计
初步设计其实就是人的脚印，给渲染到一张`RT`上，然后把`RT`作为`texture`给雪地材质用

# 实现
## 画板蓝图
首先肯定是先来做一个画板，做一个`Actor`即可
然后声明一个`RT`，其实我们只需要用高度，直接声明一个`R`通道即可，但是现在我们为了好玩开一个`RGBA16f`，k
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc0NjI1MzUwMSwtNzU1MjI0NzM1XX0=
-->