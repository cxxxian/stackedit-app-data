![输入图片说明](/imgs/2024-10-17/om9dFw5lTCDsgIYc.png)
# 顶点索引
![输入图片说明](/imgs/2024-10-17/ZjXfNWzh1J1hgAaA.png)
![输入图片说明](/imgs/2024-10-22/NHwsbx0XyickoDSs.png)
```
void prepareVAO() {
    float positions[] = {
        -0.5f, -0.5f, 0.0f,
        0.5f, -0.5f, 0.0f,
        0.0f, 0.5f, 0.0f,
        0.5f, 0.5f, 0.0f
    };
    unsigned int indices[] = {
        0, 1, 2,
        2, 1, 3
    };
    //2 VBO创建
    GLuint vbo;
    glGenBuffers(1, &vbo);
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glBufferData(GL_ARRAY_BUFFER, sizeof(positions), positions, GL_STATIC_DRAW);

    //3 EBO创建
    GLuint ebo;
    glGenBuffers(1, &ebo);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

    //4 VAO创建
    glGenVertexArrays(1, &vao);
    glBindVertexArray(vao);

    //5 绑定vbo ebo 加入属性描述信息
    //5.1 加入位置属性描述信息
    glBindBuffer(GL_ARRAY_BUFFER, vbo);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(float) * 3, (void*)0);

    //5.2 加入ebo到当前的vao
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg2NzY5MDY4OV19
-->