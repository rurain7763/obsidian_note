###### 이론
`uniform`
read-only variables
every shader reads as same value
useful for global values

`attributes`
data itself
ex. positions, normals, uvs, colors

`varyings`
output from previous shader program
values are interpolated

`sampler2D`
texture variable

`mix(a, b, t)`
get interpolated values by t
```cpp
return a + t * (b - a) or return (1 - t) * a + t * b
```

`texture2D(sampler2D, uv)`
get pixel color of texture

`step(edge, x)`
``` cpp
if(x < edge) return 0;
return 1.0;
```

`smoothstep(edge1, edge2, x)`
``` cpp
t = clamp((x - edge1) / (edge1 - edge2), 0.0, 1.0);
return t * t * (3.0 - 2.0 * t);
```

`min(x, y)`
returns the maximum of the 2 values;

`max(x, y)`
returns the minimum of the 2 values;

`clamp(v, min, max)`
clamp v to the range min and max
saturate라고 하기도 함.
```cpp
return min(max(v, min), max);
```

`abs(v)`
절댓값

`floor(v)`
내림

`ceil(v)`
올림

`round(v)`
반올림

`fract(v)`
return fraction of part v

`mod(x, y)`
return x % y
#### Homework
###### Flip the gradient
- try to flip gradient with uvs
```glsl
varying vec2 vUvs;
void main() {
	gl_FragColor = vec4(vec3(1.0 - vUvs.x), 1.0);
}
```
###### Change the colors
- try to make the top left red, and the bottom rigth blue with uvs
```glsl
varying vec2 vUvs;
void main() {
	gl_FragColor = vec4(vUvs.y, vUvs.x, 0.0, 1.0);
}
```
###### Flip texture horizontally
- flip the image horizontally
```glsl
varying vec2 vUvs;
uniform sampler2D diffuse;
void main() {
	vec4 color = texture2D(diffuse, vec2(1.0 - vUvs.x, vUvs.y))
	gl_FragColor = color;
}
```
###### Modify shader to have 3 sections
- Try changing the shader to have 3 sections instead of 2. Have the top section be a step function, the middle can be the mix function, and the bottom can be the smoothstep.
```glsl
void main() {
	vec3 color = vec3(0.0);

	vec3 red = vec3(0.7, 0.0, 0.0);
	vec3 green = vec3(0.0, 1.0, 0.0);
	vec3 blue = vec3(0.0, 0.0, 1.0);
	vec3 black = vec3(0.0);
	vec3 white = vec3(1.0);

	float div = 1.0 / 3.0;
	
	float line1 = abs(vUvs.y - div * 2.0);
	float line2 = abs(vUvs.y - div);

	float stepValue = step(0.5, vUvs.x);
	float linearValue = vUvs.x;
	float smoothStepValue = smoothstep(0.0, 1.0, vUvs.x);

	vec3 stepColor = mix(green, blue, stepValue);
	vec3 mixColor = mix(green, blue, linearValue);
	vec3 smoothStepColor = mix(green, blue, smoothStepValue);

	color = stepColor;
	color = mix(color, mixColor, 1.0 - step(div * 2.0, vUvs.y));
	color = mix(color, smoothStepColor, 1.0 - step(div, vUvs.y));
	color = mix(white, color, smoothstep(0.0, 0.005, line1));
	color = mix(white, color, smoothstep(0.0, 0.005, line2));
	color = mix(red, color, smoothstep(0.0, 0.0075, abs(vUvs.y - mix(div * 2.0, 1.0, stepValue))));
	color = mix(red, color, smoothstep(0.0, 0.0075, abs(vUvs.y - mix(div, div * 2.0, linearValue))));
	color = mix(red, color, smoothstep(0.0, 0.0075, abs(vUvs.y - mix(0.0, div, smoothStepValue))));
	
	gl_FragColor = vec4(color, 1.0);
}
```