![输入图片说明](/imgs/2024-10-14/ckJoOAGeXvYbRpas.png)
![输入图片说明](/imgs/2024-10-14/BV7yPQmKIr8AtKBf.png)
![输入图片说明](/imgs/2024-10-14/oc4XWzWYxZHbeDKC.png)
解释：
**创建一个vbo，还没有真正给分配显存**
GLuint vbo = 0;
此时vbo是0，
GL_CALL(glGenBuffers(1, &vbo));
此时vbo值为1，//**注意方法内的1是个数，和值不一样**
GL_CALL(glDeleteBuffers(1, &vbo));
销毁之后vbo就不是1了，
GLuint vboArr[] = { 0,0,0 };
GL_CALL(glGenBuffers(3, vboArr));
此时vboArr分别是1，2，3
因为上面1号vbo被销毁了所以这边会重新生成一个1号

![输入图片说明](/imgs/2024-10-15/FqvpJkOWIkLIPhhv.png)
#  单属性
![输入图片说明](/imgs/2024-10-15/VL3rESofMkzpPw2S.png)
```
void prepare() {
    float vertices[] = {
        -0.5f, -0.5f, 0.0f,
        0.5f, -0.5f, 0.0f,
        0.0f, 0.5f, 0.0f
    };
    //1 生成一个vbo
    GLuint vbo = 0;
    GL_CALL(glGenBuffers(1, &vbo));

    //2 绑定当前vbo，到opengl状态机的当前vbo插槽上
    //GL_ARRAY_BUFFER:表示当前vbo这个插槽
    GL_CALL(glBindBuffer(GL_ARRAY_BUFFER, vbo));

    //3 向当前vbo传输数据，也是在开辟显存
    GL_CALL(glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW));
}
```
# 多属性
两种方法
### 每种属性分别存储vbo
![输入图片说明](/imgs/2024-10-15/lwo4fHrIqAZABURB.png)
### 所有属性存储为一个vbo
![输入图片说明](/imgs/2024-10-15/oDOMkK7aLkssQaCZ.png)


<!--stackedit_data:
eyJoaXN0b3J5IjpbNTc1NzkwNjUsLTExMTA5OTM0NzUsLTEwNz
U1NzUwMjksMjczMjQ4NjQwLDE5Mjk5MjY5NTQsODM3OTMxMjks
LTU5MzM0ODg0OSwtMTE5MTQ0Nzg5NywtMTUzNjM2Njk1NCw2Nj
k0MzEyMTksLTIwODg3NDY2MTJdfQ==
-->