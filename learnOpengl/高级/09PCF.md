# 概念

![输入图片说明](/imgs/2025-02-26/v9skIEn2B1oTFxAx.png)

![输入图片说明](/imgs/2025-02-26/QcwymgPDZ868xQwW.png)

![输入图片说明](/imgs/2025-02-26/aoL9xIRPTHoDHuEM.png)

引入`texelSize`，这是是图素的概念，是以`uv`为单位的
以前用的`pixel`叫像素
这里使用的方法是`textureSize`，可以根据传入的`shadowMapSampler`得到大小，后面的`0`是代表`mipmap`的等级，我们一直用的是`0`
其实就是九宫格的值分别进行判断，挡住的值为`1`，没挡住为`0`，最后把九个值累加起来求平均值

![输入图片说明](/imgs/2025-02-26/g3SNfD41FKDMGYP5.png)

# 实现
去到`phongShadow.frag`创建一个计算`pcf`的方法
然后在`main`方法中，原本调用的是`caculateShadow`，现在改为使用`pcf`
```glsl
float pcf(vec3 normal, vec3 lightDir){
	 //1 找到当前像素在光源空间内的NDC坐标
	vec3 lightNDC = lightSpaceClipCoord.xyz/lightSpaceClipCoord.w;

	//2 找到当前像素在ShadowMap上的uv
	vec3 projCoord = lightNDC * 0.5 + 0.5;
	vec2 uv = projCoord.xy;
	float depth = projCoord.z;

	vec2 texelSize = 1.0 / textureSize(shadowMapSampler, 0);

	//3 遍历九宫格，每一个的深度值都需要与当前像素在光源下的深度值进行比较
	float sum = 0.0;
	for(int x = -1; x <= 1; x++){
		for(int y = -1; y <= 1; y++){
			float closestDepth = texture(shadowMapSampler, uv + vec2(x, y) * texelSize).r;
			sum += closestDepth < (depth - getBias(normal, lightDir))? 1.0:0.0;
		}
	}
	return sum / 9.0;
}

void main()
{
	...
	float shadow = pcf(normal, -directionalLight.direction);
	...
}
```
已经有点过渡的感觉了，但是阴影会呈现出一种很规律的边界，很丑

![输入图片说明](/imgs/2025-02-26/9r0eedg2AfIPoLR3.png)

# 泊松采样
引入泊松采样

![输入图片说明](/imgs/2025-02-26/KgroWOfL0LtCZeP9.png)

![输入图片说明](/imgs/2025-02-26/TPLgOdeIKnC9BCVI.png)


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MjI5NjY3NDIsMTE2MDM2MTkxNSwxNj
c2NTY1MjExLDc1NDg4MDY3NV19
-->