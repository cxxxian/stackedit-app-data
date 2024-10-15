![输入图片说明](/imgs/2024-10-14/ckJoOAGeXvYbRpas.png)
# VBO
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
##  单属性
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
## 多属性
两种方法
### 每种属性分别存储vbo
![输入图片说明](/imgs/2024-10-15/lwo4fHrIqAZABURB.png)
### 所有属性存储为一个vbo
![输入图片说明](/imgs/2024-10-15/oDOMkK7aLkssQaCZ.png)
# VAO
![输入图片说明](/imgs/2024-10-15/dYhwYhWG9BFbFDwH.png)
![输入图片说明](/imgs/2024-10-15/DWMQvAWZoH1aSj1e.png)
![输入图片说明](/imgs/2024-10-15/CEMif7RVfrOkMZCt.png)
多个VBO需要多增加一个i用来描述存储在几号VBO
![输入图片说明](/imgs/2024-10-15/af8jRJMZUcXGMppq.png)
下面两张图：位置数据可以正常读取，但是颜色信息需要知道偏移量，即前面需要偏移三个位置信息的float。
![输入图片说明](/imgs/2024-10-15/A4sSvzt46Ds1vZPW.png)

![输入图片说明](/imgs/2024-10-15/zb0yhgD4Qte5js2m.png)
## VAO总结描述
### 此处的信息可以分开放到不同的VBO也可以是同一个，只是一个例子
![输入图片说明](/imgs/2024-10-15/MGtlV336Fb4m1ZrD.png)
![输入图片说明](/imgs/2024-10-15/etymHzpqTmBzwsxg.png)
VAO中有很多个索引，index用来设置描述第几个属性
![输入图片说明](/imgs/2024-10-15/fQVGGw1Fx359HKWc.png)
![输入图片说明](/imgs/2024-10-15/yTV90sL77pcvYVUz.png)
```
//4 生成vao
    GLuint vao = 0;
    glGenVertexArrays(1, &vao);
    glBindVertexArray(vao);

    //5 分别将位置/颜色属性的描述信息加入vao当中
    glBindBuffer(GL_ARRAY_BUFFER, posVbo);//只有绑定了posVbo，下面的属性描述才会与此vbo相关
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);//将0转为指针

    glBindBuffer(GL_ARRAY_BUFFER, colorVbo);//只有绑定了posVbo，下面的属性描述才会与此vbo相关
    glEnableVertexAttribArray(1);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);

    //取消vao的bind
    glBindVertexArray(0);
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcyODUxMDk1MSwtODU3NzYyMDQxLC02Nj
AwNDUyNTMsLTE5NTgxNzMwODgsNTc1NzkwNjUsLTExMTA5OTM0
NzUsLTEwNzU1NzUwMjksMjczMjQ4NjQwLDE5Mjk5MjY5NTQsOD
M3OTMxMjksLTU5MzM0ODg0OSwtMTE5MTQ0Nzg5NywtMTUzNjM2
Njk1NCw2Njk0MzEyMTksLTIwODg3NDY2MTJdfQ==
-->