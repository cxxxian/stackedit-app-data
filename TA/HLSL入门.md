# 广告牌切换效果

![输入图片说明](/imgs/2026-07-12/WzZYFToRG5cs8eFR.png)

## 简单一句话区分
-   **TexCoord = 拿模型自带的 UV 坐标（附带横竖基础渐变），用来贴图片**
-   **LinearGradient = 凭空生成一条可自由调角度、长短、位置的黑白渐变，专门做渐变光、遮罩、进度条**
数据来源读取模型预烘焙 UV，依赖模型拓扑纯数学程序化生成，
不依赖模型 UV输出类型二维向量 (U,V)，同时包含横竖两套渐变一维标量，单方向线性渐变


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQyNjI1ODc4NSwtNjYxODc3MjZdfQ==
-->