![输入图片说明](/imgs/2024-10-17/yqGCZeQ6aaO6VyUf.png)
![输入图片说明](/imgs/2024-10-17/naV9IKOS1UMOFebi.png)
# Vertex Shader
![输入图片说明](/imgs/2024-10-17/tnBG0s3bZiGZKoZ6.png)
### layout
![输入图片说明](/imgs/2024-10-17/XdjVYCXNYkkT1JIg.png)
### gl_Position
此处我们最终目的就是输出NDC坐标，此处由于我们的输入就已经是NDC坐标了，如果不是的话需要通过一些矩阵变换
![输入图片说明](/imgs/2024-10-17/sZt9j4e6kVbj5bgz.png)
# Fragment Shader
![输入图片说明](/imgs/2024-10-17/ZUnSyAWn1WHGSJ1q.png)
# Shader的编译
![输入图片说明](/imgs/2024-10-17/1ZoqeqyEmXG5gGWK.png)
此处的glShaderSource

 1. const GLchar *const* string
 2. count用来记录String数组共有多少个字符串

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgyNzY2NTg3NiwtODY3OTgzMjQ2LDQyNj
ExNzg4OV19
-->