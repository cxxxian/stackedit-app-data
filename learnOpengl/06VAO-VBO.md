![输入图片说明](/imgs/2024-10-14/ckJoOAGeXvYbRpas.png)
![输入图片说明](/imgs/2024-10-14/BV7yPQmKIr8AtKBf.png)
![输入图片说明](/imgs/2024-10-14/oc4XWzWYxZHbeDKC.png)
解释：
GLuint vbo = 0;
此时vbo是0，
GL_CALL(glGenBuffers(1, &vbo));
此时vbo值为1，
GL_CALL(glDeleteBuffers(1, &vbo));
销毁之后vbo就bu
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjM5MjA1NDE4LC0xMTkxNDQ3ODk3LC0xNT
M2MzY2OTU0LDY2OTQzMTIxOSwtMjA4ODc0NjYxMl19
-->