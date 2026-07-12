# 广告牌切换效果

![输入图片说明](/imgs/2026-07-12/WzZYFToRG5cs8eFR.png)

## 简单一句话区分
**TexCoord** = 
1. 拿模型自带的 UV 坐标（附带横竖基础渐变），用来贴图片
2. 数据来源读取模型预烘焙 UV
3. 不依赖模型 UV输出类型二维向量 (U,V)

**LinearGradient** = 
1. 凭空生成一条可自由调角度、长短、位置的黑白渐变，专门做渐变光、遮罩、进度条
2. 依赖模型拓扑纯数学程序化生成
3. 同时包含横竖两套渐变一维标量，单方向线性渐变

![输入图片说明](/imgs/2026-07-12/fhjTOWpFqpl9ELxM.png)

利用`TexCoord * 10 * ceil`，可以讲原本`0-1`的`uv`转成`0-10`，然后用`ceil`之后就能实现上面这种画格子的效果，
但是因为`TexCoord * 10`的原因，整个颜色输出会变亮，所以我们在`ceil`之后要手动`divide`除以`10`
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEzODYwMzM0NSwtNjYxODc3MjZdfQ==
-->