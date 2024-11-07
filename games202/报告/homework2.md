
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

  

/*

* Evaluate directional light with shadow map

* uv is in screen space, [0, 1] x [0, 1].

*

*/

//light项

vec3 EvalDirectionalLight(vec2 uv) {

//GetGBufferuShadow(uv)获取当前像素位置的阴影信息，表示当前点是否被阴影遮挡

vec3 Le = GetGBufferuShadow(uv) * uLightRadiance;

return Le;

}
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2OTExMjY1MzNdfQ==
-->