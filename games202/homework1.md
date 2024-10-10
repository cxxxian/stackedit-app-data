```
CalcLightMVP(translate, scale) {
let lightMVP = mat4.create();
let modelMatrix = mat4.create();
let viewMatrix = mat4.create();
let projectionMatrix = mat4.create();
//Edit Start
// Model transform
//处理物体本身在场景中的位置和大小（模型的世界空间坐标）
mat4.translate(modelMatrix, modelMatrix, translate)
mat4.scale(modelMatrix, modelMatrix, scale)
// View transform
//this.lightPos 是光源的位置，this.focalPoint 是光源看向的目标点
mat4.lookAt(viewMatrix, this.lightPos, this.focalPoint, this.lightUp)
// Projection transform
var r = 100;
var l = -r;
var t = 100;
var b = -t;
var n = 0.01;
var f = 200;
//投影矩阵将物体从光源视角投影到一个正交投影空间
mat4.ortho(projectionMatrix, l, r, b, t, n, f);

//Edit End

//首先将投影矩阵和视图矩阵相乘，生成从世界空间到光源正交投影空间的变换矩阵
mat4.multiply(lightMVP, projectionMatrix, viewMatrix);

//然后将模型矩阵乘上这个结果，完成从物体本地坐标到光源投影空间的变换
mat4.multiply(lightMVP, lightMVP, modelMatrix);

//lightMVP 矩阵最终用于在渲染阴影贴图时，将每个物体从其本地坐标转换到光源的投影空间
return lightMVP;

}
```
这个函数生成的 `lightMVP` 矩阵实际上是用于光源视角的模型-视图-投影变换矩阵，主要用于**生成阴影贴图（Shadow Map）**。通过这个矩阵，场景中的每个物体从光源的角度被投影到一个正交投影空间中，生成一个深度贴图，用于模拟物体之间的阴影关系。下面我详细解释它的作用以及为什么它适用于阴影贴图的生成。

### 光源生成阴影贴图的背景

在实时渲染中，生成阴影贴图的过程通常分为两个步骤：
1. **从光源角度渲染场景**：我们要模拟光源看到的场景，记录从光源出发到场景中每个物体的最远距离（深度信息）。这些信息存储在阴影贴图中。
2. **在相机渲染场景时使用阴影贴图**：根据相机看到的每个像素，参考阴影贴图来判断它是否在阴影中。

### `lightMVP` 矩阵的作用

该矩阵让我们可以从光源的角度将整个场景投影到一个平面上，这样可以生成一个阴影贴图，记录光源视角下的深度信息。它结合了模型、视图、投影三种变换矩阵。

### 矩阵变换的具体意义

1. **模型矩阵（Model Matrix）**：
   ```javascript
   mat4.translate(modelMatrix, modelMatrix, translate);
   mat4.scale(modelMatrix, modelMatrix, scale);
   ```
   - 这个变换处理物体本身在场景中的位置和大小（模型的世界空间坐标）。
   - 在阴影贴图中，我们需要将场景中的物体按其实际位置和大小投影到光源的视角下，所以模型矩阵不可或缺。

2. **视图矩阵（View Matrix）**：
   ```javascript
   mat4.lookAt(viewMatrix, this.lightPos, this.focalPoint, this.lightUp);
   ```
   - 视图矩阵定义了**光源的位置和方向**。就像相机有一个视图矩阵一样，光源也有一个视图矩阵。这个矩阵将场景从世界坐标系转换为光源视角的坐标系。
   - `this.lightPos` 是光源的位置，`this.focalPoint` 是光源看向的目标点，而 `this.lightUp` 定义了光源的上方向。
   - 当我们从光源的角度投影场景时，这个矩阵确保所有物体相对于光源被正确定位。

3. **投影矩阵（Projection Matrix）**：
   ```javascript
   var r = 100;
   var l = -r;
   var t = 100;
   var b = -t;
   var n = 0.01;
   var f = 200;
   mat4.ortho(projectionMatrix, l, r, b, t, n, f);
   ```
   - 投影矩阵将物体从光源视角投影到一个正交投影空间。正交投影适合于**方向光**（如太阳光）这样的光源，因为它们的光线是平行的，不需要透视投影。
   - 这里的参数（`r`, `l`, `t`, `b`, `n`, `f`）定义了正交视锥体的边界，即光源照射的区域。`n` 是近裁剪面，`f` 是远裁剪面，定义了光源能照射的最远和最近的范围。
   - 通过这个投影矩阵，光源的视锥体内的所有物体都会被投影到二维平面上，生成阴影贴图。

### 矩阵乘法

```javascript
mat4.multiply(lightMVP, projectionMatrix, viewMatrix);
mat4.multiply(lightMVP, lightMVP, modelMatrix);
```
- 首先将投影矩阵和视图矩阵相乘，生成从世界空间到光源正交投影空间的变换矩阵。
- 然后将模型矩阵乘上这个结果，完成从物体本地坐标到光源投影空间的变换。
- 这个 `lightMVP` 矩阵最终用于在渲染阴影贴图时，将每个物体从其本地坐标转换到光源的投影空间。

### 为什么它用于生成 ShadowMap

- 阴影贴图记录的是从光源角度看到的场景深度。为了生成阴影贴图，我们需要从光源的角度来看场景，而 `lightMVP` 矩阵正是将场景从光源的视角投影到一个正交空间中。
- 当我们从光源视角渲染场景时，我们只关心场景中物体的深度信息，`lightMVP` 矩阵能够将场景中的每个物体的深度信息正确地投影到二维空间，从而生成阴影贴图。
  
简而言之，`lightMVP` 矩阵让我们能够从光源的角度渲染场景，这个过程会生成一个深度图，这个图在后续渲染中用来判断哪些像素点处于阴影中。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMzE2ODI1NzddfQ==
-->