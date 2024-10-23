# GLSL
![输入图片说明](/imgs/2024-10-23/qlG2osoa38B3S6Td.png)
![输入图片说明](/imgs/2024-10-23/vXuEc12KrJwJBGhd.png)
![输入图片说明](/imgs/2024-10-23/V7oV4iQX0whKxbHO.png)
# shader变量
![输入图片说明](/imgs/2024-10-23/kr29NmybGr3lCW5x.png)
## 输入
![输入图片说明](/imgs/2024-10-23/WCQvMQcVIne89dCH.png)
此方法可以动态获取属性编号，但是这样的话需要将mProgram公开（不推荐）
![输入图片说明](/imgs/2024-10-23/1DC6tig6fG5BKJyH.png)
属性编号是从上到下分配的
两种顺序会得到不同的location值
posLocation = 0
colorLocation = 1
```
in vec3 aPos;
in vec3 aColor;
```
posLocation = 1
colorLocation = 0
```
in vec3 aColor;
in vec3 aPos;
```
## uniform
![输入图片说明](/imgs/2024-10-23/opnmygIOSyWXHtDl.png)
![输入图片说明](/imgs/2024-10-23/03DSXGjREaSWkO1J.png)
![输入图片说明](/imgs/2024-10-23/im4pWGswALBlNcNu.png)
![输入图片说明](/imgs/2024-10-23/uZS1RM7F7Dry3u65.png)
eg.
```
uniform int a;
glUniform1i(。。。);
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkxODEzMDkwMiwtMTM0MDc2ODk0NSwtMz
I4MzgxNDY1LDQ2MjYzMjA5NCwyMDk1MDY2MDQ3LDE3OTc4NTUw
NTJdfQ==
-->