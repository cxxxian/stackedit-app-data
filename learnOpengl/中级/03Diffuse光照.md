在main.cpp中声明光照的参数，然后将这些参数搞到shader中
```
//平行光：参数（方向、光强）uniform变量形式
glm::vec3 lightDirection = glm::vec3(-1.0f, -1.0f, -1.0f);
glm::vec3 lightColor = glm::vec3(0.9f, 0.85f, 0.75f);
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk2NTEyMjQ0NCwtMTcyNTUyMjU4NSwtMj
A4ODc0NjYxMl19
-->