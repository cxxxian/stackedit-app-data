
# GPU为什么不适合用if或者switch
这个问题关键在于 GPU 的执行模型，叫做 **SIMD**（Single Instruction, Multiple Data）或 **SIMT**（NVIDIA 的变种：Single Instruction, Multiple Threads）

### GPU 是“锁步执行”的 —— 一组线程（Warp/Wavefront）执行同一条指令

> 在 GPU 上，32 或 64 个线程是打包在一起执行的（一个 Warp/Wave），他们 **必须执行同一条指令**。

#### 如果出现 `if` 分支会发生什么？

```cpp
if (a > 0) {     
	color = red; 
} else {     
	color = blue; 
}
```

![输入图片说明](/imgs/2025-07-14/Tj3ZjRISqyVcSAzq.png)

### 正常我们以为会怎样执行？

我们希望每个线程直接走它自己的分支，互不影响，对吧？

但 GPU **不行**，因为这 4 个线程是在同一个 Warp 中，它们**必须执行一模一样的指令流**！

----------

### 实际 GPU 会怎么做？

1.  **先执行 if 分支：**
    
    -   所有线程**都执行** `color = red;`
        
    -   但 GPU 会“屏蔽”不该执行的线程（1 和 3）→ 它们执行这条指令但不写结果
        
2.  **再执行 else 分支：**
    
    -   所有线程**再执行一次** `color = blue;`
        
    -   这次屏蔽掉线程 0 和 2，只让 1 和 3 写入结果

可以改写为：
`mix` 线性插值写法（推荐）

```glsl
vec3 red  = vec3(1.0, 0.0, 0.0);
vec3 blue = vec3(0.0, 0.0, 1.0);

float t   = step(0.5, a);          // a ≤ 0.5 → 0，a > 0.5 → 1
vec3 color = mix(blue, red, t);    // 等价于 if/else
```
-   `step(edge, x)` 在硬件里是无分支的比较；
-   `mix(x, y, t)` = `x * (1‑t) + y * t`，同样无分支。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI3NTg3ODIyNF19
-->