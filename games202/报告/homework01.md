# games202-实验报告-01

## 任务0：实验环境
-   电脑品牌：惠普
-   电脑型号：暗影精灵8pro
-   CPU型号：12th Gen Intel(R) Core(TM) i7-12700H
    -   CPU核心（core）数量：4
    -   CPU默频/睿频：2.30 GHz
-   内存规格：// 例如：DDR2-400
-   内存容量：512G
-   存储类型：SSD
-   显卡型号：Nvidia GeForce RTX 3060
-   屏幕分辨率：1920*1080
-   最大功率：280W
-   操作系统：Windows11

## 任务1：Shadow Map
完善DirectionalLight中的CalcLightMVP(translate, scale)函数
```
CalcLightMVP(translate, scale) {
let lightMVP = mat4.create();
let modelMatrix = mat4.create();
let viewMatrix = mat4.create();
let projectionMatrix = mat4.create();
// Model transform
//translate(输出的矩阵，处理的矩阵，vec3矩阵)
mat4.translate(modelMatrix, modelMatrix, translate);
mat4.scale(modelMatrix, modelMatrix, scale);
// View transform
//参数在engine定义
//let lightPos = [0, 80, 80];
//let focalPoint = [0, 0, 0];
//let lightUp = [0, 1, 0]
//focalPoint是lookat的方向
mat4.lookAt(viewMatrix, this.lightPos, this.focalPoint, this.lightUp);
// Projection transform
//n 是近裁剪面，f 是远裁剪面
var r = 100;
var l = -100;
var t = 100;
var b = -100;
var n = 1e-2;
var f = 400;
mat4.ortho(projectionMatrix, l, r, b, t, n, f);
mat4.multiply(lightMVP, projectionMatrix, viewMatrix);
mat4.multiply(lightMVP, lightMVP, modelMatrix);
return lightMVP;
}
```
## 任务2：调试示例（DebugDemo）

-   IDEA中以下功能的热键：
    -   构建项目：Ctrl+F9
    -   运行程序：
    -   调试程序：
-   IDEA中以下调试功能的热键、用途：
    -   Step Over:
    -   Step Into:
    -   Step Out:
    -   Run to Cursor:
    -   Resume Program:
    -   Stop Program:
-   断点（Breakpoints）的用途？IDEA下断点的热键（从Run菜单中找）

## 任务3：修正程序（Fixme）

-   简要解释下列概念并各举一个例子（程序片段用反引号包围，例如`1+2`）。
    -   词法错误：
    -   语法错误：
    -   语义错误：
    -   逻辑错误：

## 任务4（选做）：命令行

-   环境变量有什么用途？
-   `PATH`环境变量的作用？
-   JDK的`bin`目录中以下命令的用途？（本项可上网搜索，不查重）
    -   `jar`：打包。
    -   `java`：
    -   `javac`：
    -   `jdb`：
    -   `jps`：
    -   `jshell`：
    -   `jstack`：
    -   `jstat`：

## 实验总结

-   请简述实验的心得体会。欢迎对实验形式、内容提出意见和建议。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2MDc0NzgyNzldfQ==
-->