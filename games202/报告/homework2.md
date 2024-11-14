## 实现对场景直接光照的着色(考虑阴影)
在src\shaders\ssrShader\ssrFragment.glsl
```
//BRDF项
vec3 EvalDiffuse(vec3 wi, vec3 wo, vec2 uv) {
//当前像素（屏幕空间坐标 uv 对应的点）的漫反射颜色
vec3 albedo = GetGBufferDiffuse(uv);
//当前像素的法线
vec3 normal = GetGBufferNormalWorld(uv);
float cos = max(0., dot(normal, wi));
//INV_PI是1/Π
return albedo * cos * INV_PI;
}
//light项
vec3 EvalDirectionalLight(vec2 uv) {
//GetGBufferuShadow(uv)获取当前像素位置的阴影信息，表示当前点是否被阴影遮挡
vec3 Le = GetGBufferuShadow(uv) * uLightRadiance;
return Le;
}
```
根据渲染方程解出直接光照信息
```
void main() {
float s = InitRand(gl_FragCoord.xy);
vec3 L = vec3(0.0);
vec3 worldPos = vPosWorld.xyz;
vec2 screenUV = GetScreenCoordinate(vPosWorld.xyz);
vec3 wi = normalize(uLightDir);
vec3 wo = normalize(uCameraPos - worldPos);
//L = GetGBufferDiffuse(GetScreenCoordinate(vPosWorld.xyz));
// 直接光照
L = EvalDiffuse(wi, wo, screenUV) * EvalDirectionalLight(screenUV);
vec3 color = pow(clamp(L, vec3(0.0), vec3(1.0)), vec3(1.0 / 2.2));
gl_FragColor = vec4(vec3(color.rgb), 1.0);
}
```
## Screen Space Ray Tracing
RayMarch目的是求光线与物体交点，原理就是模拟光线从给定一个起点沿着某个方向每次步进一定的距离，用步进后光线的深度对比光线所在的屏幕坐标的场景物体深度，若光线深度大于场景物体深度，则相交，实现如下：
(需要设定最大步进次数，避免不相交时计算没有退出条件的问题，另一方面也可以把RayMarch的性能消耗在一定程度上做限制，步进太远还没有交点时，就认为没有交点。)
```
bool RayMarch(vec3 ori, vec3 dir, out vec3 hitPos) {
	//ori: 光线的起始位置。
	//dir: 光线的方向，一个单位向量。
	float step = 0.05;
	//光线行进的最大次数
	const int totalStepTimes = 150;
	int curStepTimes = 0;
	vec3 stepDir = normalize(dir) * step;
	vec3 curPos = ori;
	for(int curStepTimes = 0; curStepTimes < totalStepTimes; curStepTimes++)
	{
		//将当前光线位置 curPos 转换为屏幕空间坐标
		vec2 screenUV = GetScreenCoordinate(curPos);
		//获取当前光线位置在深度图中的深度值
		float rayDepth = GetDepth(curPos);
		//获取 G-Buffer 中对应位置的深度值
		float gBufferDepth = GetGBufferDepth(screenUV);
		if(rayDepth - gBufferDepth > 0.0001){
			hitPos = curPos;
			return true;
		}
		curPos += stepDir;
	}
	return false;
}
```
一个专门用来测试SSR的函数。
```
// test Screen Space Ray Tracing 
vec3 EvalReflect(vec3 wi, vec3 wo, vec2 uv) {
  vec3 worldNormal = GetGBufferNormalWorld(uv);
  vec3 relfectDir = normalize(reflect(-wo, worldNormal));
  vec3 hitPos;
  if(RayMarch(vPosWorld.xyz, relfectDir, hitPos)){
      vec2 screenUV = GetScreenCoordinate(hitPos);
      return GetGBufferDiffuse(screenUV);
  }
  else{
    return vec3(0.); 
  }
}
```
最后在main函数中把之前实现的直接光照换成要测SSR的函数
```
//ssrFragment.glsl
void main() {
  float s = InitRand(gl_FragCoord.xy);
  vec3 L = vec3(0.0);
  // L = GetGBufferDiffuse(GetScreenCoordinate(vPosWorld.xyz));
  vec3 worldPos = vPosWorld.xyz;
  vec2 screenUV = GetScreenCoordinate(vPosWorld.xyz);
  vec3 wi = normalize(uLightDir);
  vec3 wo = normalize(uCameraPos - worldPos);
  // 直接光照
  // L = EvalDiffuse(wi, wo, screenUV) * EvalDirectionalLight(screenUV);
  // Screen Space Ray Tracing 的反射测试
  L = (GetGBufferDiffuse(screenUV) + EvalReflect(wi, wo, screenUV))/2.;
  vec3 color = pow(clamp(L, vec3(0.0), vec3(1.0)), vec3(1.0 / 2.2));
  gl_FragColor = vec4(vec3(color.rgb), 1.0);
}
```
往main中加入for循环启用RayMarch效果
```
#define SAMPLE_NUM 1

void main() {
  float s = InitRand(gl_FragCoord.xy);
  vec3 L = vec3(0.0);
  // L = GetGBufferDiffuse(GetScreenCoordinate(vPosWorld.xyz));

  vec3 worldPos = vPosWorld.xyz;
  vec2 screenUV = GetScreenCoordinate(vPosWorld.xyz);
  vec3 wi = normalize(uLightDir);
  vec3 wo = normalize(uCameraPos - worldPos);

  // 直接光照
  L = EvalDiffuse(wi, wo, screenUV) * EvalDirectionalLight(screenUV);

  // Screen Space Ray Tracing 的反射测试
  // L = (GetGBufferDiffuse(screenUV) + EvalReflect(wi, wo, screenUV))/2.;

  vec3 L_ind = vec3(0.0);
  for(int i = 0; i < SAMPLE_NUM; i++){
    float pdf;
    vec3 localDir = SampleHemisphereCos(s, pdf);
    vec3 normal = GetGBufferNormalWorld(screenUV);
    vec3 b1, b2;
    LocalBasis(normal, b1, b2);
    vec3 dir = normalize(mat3(b1, b2, normal) * localDir);

    vec3 position_1;
    if(RayMarch(worldPos, dir, position_1)){
      vec2 hitScreenUV = GetScreenCoordinate(position_1);
      L_ind += EvalDiffuse(dir, wo, screenUV) / pdf * EvalDiffuse(wi, dir, hitScreenUV) * EvalDirectionalLight(hitScreenUV);
    }
  }

  L_ind /= float(SAMPLE_NUM);

  L = L + L_ind;

  vec3 color = pow(clamp(L, vec3(0.0), vec3(1.0)), vec3(1.0 / 2.2));
  gl_FragColor = vec4(vec3(color.rgb), 1.0);
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA1NzIwOTQ3NywtNTM0MDU1Mjc4LDc2NT
M3MTcwNV19
-->