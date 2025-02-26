![输入图片说明](/imgs/2025-02-26/v9skIEn2B1oTFxAx.png)

![输入图片说明](/imgs/2025-02-26/QcwymgPDZ868xQwW.png)

![输入图片说明](/imgs/2025-02-26/aoL9xIRPTHoDHuEM.png)

引入`texelSize`，这是是图素的概念，是以`uv`为单位的
以前用的`pixel`叫像素
这里使用的方法是`textureSize`，可以根据传入的`shadowMapSampler`得到大小，后面的`0`是代表`mipmap`的等级，我们一直用的是`0`
其实就是九宫格的值分别进行判断，挡住的值为`1`，没挡住为`0`，最后把九个值累加起来求平均值

![输入图片说明](/imgs/2025-02-26/g3SNfD41FKDMGYP5.png)

3
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk3OTU4MTMyLDE2NzY1NjUyMTEsNzU0OD
gwNjc1XX0=
-->