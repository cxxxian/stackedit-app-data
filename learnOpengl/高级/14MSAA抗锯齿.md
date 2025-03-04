# 概念

![输入图片说明](/imgs/2025-03-04/B9n1v5GEoutVtS8l.png)

![输入图片说明](/imgs/2025-03-04/OWjZqnQNfpq7Babh.png)

![输入图片说明](/imgs/2025-03-04/jJFBgc4xG0urG1sN.png)

意思就是，`fragment`不会变大，变大的是针对每一个`fragment`的采样点

![输入图片说明](/imgs/2025-03-04/8sPQGq21hNJl78ji.png)

 以上就是`MSAA`过程，最后的`PostProcessPass`其实就是我们做的后处理`Gamma`校正
 
# 代码分析

![输入图片说明](/imgs/2025-03-04/IP1xRacTyMcXlFWJ.png)

![输入图片说明](/imgs/2025-03-04/T1TQCBtJe8F4dACp.png)

![输入图片说明](/imgs/2025-03-04/M3fj6hqjoIRxpDEu.png)

# 实践
## 1 创建MSAA专用纹理
```
`
## 2 创建MSAA专用FBO
## 3 Renderer中增加msaaResolve函数
## 4 绘制流程更改
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjkzMTYzNzUyLDExODQ3ODk5MjgsOTk1ND
QxMTQ2LC0xNjcxMjc0NDE3LC0xNjAxNDUyNjQ2LDExMDcyMjc2
MDksLTEwNzA0ODI2MDldfQ==
-->