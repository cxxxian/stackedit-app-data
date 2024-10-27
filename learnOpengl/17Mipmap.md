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
![输入图片说明](/imgs/2024-10-27/iPDOZZqgzvfNfVOK.png)
使用同一张图片（data），只改变width和height，比如一张`256*256`的图片，width和height传入`128*128`，这样OpenGL会只选择左下角那块`128*128`的部分
## 手动生成mipmap层级
![输入图片说明](/imgs/2024-10-27/fCyrCrJ96z4eDn6R.png)
## 三角形随时间变小
![输入图片说明](/imgs/2024-10-27/EY9G9rShjdL29bn1.png)
在texture的构造函数中
将原本的glTexImage2D改为
```
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
```
自己设置级别的level。此做法用来：遍历每个mipmap的层级，为每个级别的mipmap填充图片数据
```
for (int level = 0; true; ++level) {
        //1 将当前级别的mipmap对应的数据发往gpu端
        glTexImage2D(GL_TEXTURE_2D, level, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
        //2 判断是否退出循环
        if (width == 1 && height == 1) {
            break;
        }
        //3 计算下一次循环的宽度/高度，除二
        width = width > 1 ? width / 2 : 1;
        height = height > 1 ? height / 2 : 1;

    }
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI4ODQyMDk5MywyNTQ0NjE5ODQsMTQyNT
MzMzcxMywtNzI5ODc2NzE5LDYyMTM4NDkyNiwtNDQxMzk2MTM5
LC02NTUwODg2NjUsLTQzODY1OTg1MSwtNjExMTYwMywxMjc5Mj
kwNTQ2LC00NTc4MTg1NzIsLTEwNDE5ODAwNjAsLTEwMzYyNjM2
NzksLTE0ODU3NzY4NjQsLTEwMTMyNTIwNDMsMjA3NzQ3MjI5OF
19
-->