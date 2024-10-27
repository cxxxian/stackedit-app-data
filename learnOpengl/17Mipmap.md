![输入图片说明](/imgs/2024-10-27/HXXMESHfOhzZ7TBM.png)
![输入图片说明](/imgs/2024-10-27/VpRKtIXh3LdjQwQh.png)

# Mipmap生成方式
![输入图片说明](/imgs/2024-10-27/WnH5md67UslU9Ph9.png)
![输入图片说明](/imgs/2024-10-27/zNaiuYKYFVzpaBiy.png)
![输入图片说明](/imgs/2024-10-27/EziP1YDJyyyrkBO7.png)
![输入图片说明](/imgs/2024-10-27/dxILoFRLHn5MUZLw.png)
![输入图片说明](/imgs/2024-10-27/3IbcSjWPIiMp2X5f.png)
![输入图片说明](/imgs/2024-10-27/M8mTNQbkii610qBC.png)
![输入图片说明](/imgs/2024-10-27/baSgPHlSpSA93fhy.png)
![输入图片说明](/imgs/2024-10-27/YxneQRLSI775uahU.png)
![输入图片说明](/imgs/2024-10-27/OcdNimEhaKxJ7PV2.png)
此处用dot计算是因为贴图可能是歪的，如果是正的那当然可以直接比较dx和dy（例如（10，0）和（20，0）之间相差10）之间的差值，但是歪的话，dx可能值会是（10，30）这种。需要dot计算出斜边长度
![输入图片说明](/imgs/2024-10-27/QUOyuXHB9AnLKcMB.png)![输入图片说明](/imgs/2024-10-27/VcKIOGj5WaT26vHV.png)
# 实现
![输入图片说明](/imgs/2024-10-27/HNG4N1yGQg4cfMdG.png)
**规定**：
使用除了0之外的level，OpenGL认定你开启使用了Mipmap，必须准备好一直到`1*1`的层级，否则会报错
即是最后生成了`2*1`的图片，也需要再生成一个`1*1`的


<!--stackedit_data:
eyJoaXN0b3J5IjpbNjIxMzg0OTI2LC00NDEzOTYxMzksLTY1NT
A4ODY2NSwtNDM4NjU5ODUxLC02MTExNjAzLDEyNzkyOTA1NDYs
LTQ1NzgxODU3MiwtMTA0MTk4MDA2MCwtMTAzNjI2MzY3OSwtMT
Q4NTc3Njg2NCwtMTAxMzI1MjA0MywyMDc3NDcyMjk4XX0=
-->