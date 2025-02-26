![输入图片说明](/imgs/2025-02-26/aL4BSM1rT8dVSror.png)

这里设计的几个参数，解释几个看起来比较抽象的
当场景中的灯多起来后，就需要多个`shadowMap`，所以`camera, renderTarget`就要设计在`shadow`类下，每一个不同阴影都要有自己的`camera`和`renderTarget`

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwOTU1MDgxMDRdfQ==
-->