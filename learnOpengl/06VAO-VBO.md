![输入图片说明](/imgs/2024-10-14/ckJoOAGeXvYbRpas.png)
![输入图片说明](/imgs/2024-10-14/BV7yPQmKIr8AtKBf.png)
![输入图片说明](/imgs/2024-10-14/oc4XWzWYxZHbeDKC.png)
解释：
**创建一个vbo，还没有真正给分配显存**
GLuint vbo = 0;
此时vbo是0，
GL_CALL(glGenBuffers(1, &vbo));
此时vbo值为1，
GL_CALL(glDeleteBuffers(1, &vbo));
销毁之后vbo就不是1了，
GLuint vboArr[] = { 0,0,0 };
GL_CALL(glGenBuffers(3, vboArr));
此时vboArr分别是1，2，3
因为上面1号vbo被销毁了所以这边会重新生成一个1号

![输入图片说明](/imgs/2024-10-15/FqvpJkOWIkLIPhhv.png)
![输入图片说明](/imgs/2024-10-15/VL3rESofMkzpPw2S.png)

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjczMjQ4NjQwLDE5Mjk5MjY5NTQsODM3OT
MxMjksLTU5MzM0ODg0OSwtMTE5MTQ0Nzg5NywtMTUzNjM2Njk1
NCw2Njk0MzEyMTksLTIwODg3NDY2MTJdfQ==
-->