# 概念

![输入图片说明](/imgs/2025-02-25/bt5KiPgBKRaSixIZ.png)

从光源相机视角：
`shadowMap`由于可能分辨率不够，玩家视角的三个像素对应到`shadowMap`可能是同一个像素，（也可能玩家视角离得近，自然看到的像素就更多）
但是`shadowMap`肯定没办法记录那么多个，所以会选取中间那个进行存储，
那么从玩家视角下，很明显深度`d0 > d1 > d2`，所以`d0`会被误认为遮挡

![输入图片说明](/imgs/2025-02-25/ah8tjNTdtm30xfea.png)

所以`bias`不是一个固定值，可以暴露出来给美术人员调整，不同场景需要不同的`bias`

# 代码实现
## 1.sh
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNjg2OTYyNjAsLTEyMjMxODc5NjIsLT
IwODg3NDY2MTJdfQ==
-->