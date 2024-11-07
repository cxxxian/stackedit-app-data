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
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzY1MzcxNzA1XX0=
-->