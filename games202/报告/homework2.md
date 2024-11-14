## 实现对场景直接光照的着色(考虑阴影)
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
RayMarch目的是求光线与物体交点，原理就是模拟光线从给定一个起点沿着某个方向每次步进一定的距离，用步进后光线的深度对比光线所在的屏幕坐标的场景物体深度，若光线深度大于场景物体深度，则相交，实现如下：
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk3NzgxNDE2NSw3NjUzNzE3MDVdfQ==
-->