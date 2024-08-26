# IK重定向动画
1.先创建IK绑定，分别创建源人物与目标人物
![输入图片说明](/imgs/2024-08-09/FSRW6IMaQbl6Xb8Z.png)
2.设置重定向根
![输入图片说明](/imgs/2024-08-09/TuKwosKGRLsiZ1ZT.png)
3.新建root链
![输入图片说明](/imgs/2024-08-09/xrTR6Y74zFsp0WJS.png)
![输入图片说明](/imgs/2024-08-09/CGV2LtmZeiVDFcG2.png)
4.将各个骨骼分别建出对应的链
5.最后建立IK重定向器将两个人物进行重定向
![输入图片说明](/imgs/2024-08-09/aEkc6Xdu6oqhpy5h.png)
6.可以手动调节姿势，防止源人物与目标人物初始姿势不符合导致目标人物动画出错
![输入图片说明](/imgs/2024-08-10/BByvwsYhdxS7LKeP.png)
## 当有根运动时
导出时需要选择Root链
并在细节面板中的平移模式选择全局缩放
![输入图片说明](/imgs/2024-08-26/DoRq5STjcc6MLgnF.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQzNTExMTY0OSwxNDc4ODQ5OTYzLC0xMT
A3NjIwNjIzLDc3NTU5OTU3MywtODQ1MTcwMzA4XX0=
-->