#### Vertex array buffer
정점의 정보를 담고 있는 배열
###### 생성 및 삭제
```cpp
glm::vec3 vertices[1];
vertices[0] = glm::vec3(0.0f, 0.0f, 0.0f); // 정점 하나만 생성

glGenBuffers(1, &vbo); // 버퍼 생성
glBindBuffer(GL_ARRAY_BUFFER, vbo); // 버퍼 바인드
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); // 데이터 넘기기

//...

glDeleteBuffers(1, &vbo); // 버퍼 삭제
```
###### 사용
```cpp
glBindBuffer(GL_ARRAY_BUFFER, vbo); // 버퍼 바인드
glEnableVertexAttribArray(0); // 셰이더에서 전달 받을 변수의 번호
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0); // 현재 배열을 어떻게 해석해야 하는지 설정한다

// glDrawArrays(...)

glDisableVertexAttribArray(0);
```
#### VAO
**매 프레임마다** GL_ARRAY_BUFFER에 VBO를 연결(bind)하고, 데이터를 저장하고, Vertex Attribute들을 설정하는 것은 **비효율적**. 
**위의 모든 작업을 하나의 state로 묶어서 OpenGL Object에 저장**. 그렇게 된다면 우리는 위의 작업을 한 번만 하고, 삼각형을 그리기 전에 위 작업을 하나로 묶은 state를 current state로 만들어주기만 하면 됨. 
이 때 필요한 것이 **Vertex Array Object**(VAO).
###### 생성 및 삭제
```cpp
glGenVertexArrays(1, &_vao);

/...

glDeleteVertexArrays(1, &_vao);
```
###### 사용
```cpp
glBindVertexArray(_vao); // 바인드

// ex)
// glBindBuffer(GL_ARRAY_BUFFER, vbo);
// glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// glEnableVertexAttribArray(0); 
// glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
// ...

glBindVertexArray(0); // 언바인드
```
#### Uniform keyword
모든 셰이더 프로그램에 공통적으로 들어가는 값(레스터라이징 하지 않은 값)
```glsl
uniform int v;
unifrom float f;
uniform mat4 mat;
```
###### 전달
```cpp
glUniform1f(GetUniformLocation(name), v);
glUniformMatrix4fv(GetUniformLocation(name), 1, GL_TRUE, &v[0][0]); // 3번째 인자의 경우 column-major 인지 row-major인지 알려주는 인자(GL_TRUE의 경우 row-major)
```
#### Out keyword
다음 셰이더 프로그램의 input으로 전달한다.
``` glsl
// vs
out vec4 color; // 다음 프로그램에 전달.
void main() {
	color = ...;
	gl_Position = ...;
}

// fs
in vec4 color; // 전 프로그램으로 부터 받은 값(rasterize 된 값임)
out vec4 fragColor;
void main() {
	fragColor = color;
}
```
#### Indexed Draw
삼각형이 많아질 수록 공유하는 점이 많아짐.
그럴때마다 같은 점을 GPU 메모리에 전달하면 메모리 낭비가 심함.
따라서 해당 점의 index값들을 전달하여 공유하는 점의 경우 같은 index를 전달하여 그리게 함.

cf) OpenGL은 기본적으로 반시계방향을 앞으로 취급한다. (바꾸고 싶을 경우 `glFrontFace(GL_CW or GL_CCW)`)
###### 생성 및 삭제
```cpp
glGenBuffers(1, &_ibo);

// ...

glDeleteBuffers(1, &_ibo);
```
###### 사용
```cpp
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, _ibo);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, count * sizeof(GLuint), data, GL_STATIC_DRAW); // 데이터 전달

// ...

glDrawElements(GL_TRIANGLES, _ibo->GetCount(), GL_UNSIGNED_INT, 0); // 그리기
```
#### Texturing
#### Blending
두 가지의 색이 곂치면 어떻게 색을 결정할 것인가에 대한 내용
`glEnable(GL_Blend)` : blending 절차를 켠다
`glBlendFunc(src, dest)` : src에 얼마를 곱하고, dest에 얼마를 곱하는지 결정하는 함수. (default src = GL_ONE, dest = GL_ZERO)
cf) src는 들어오는 색, dest는 기존에 있는 색
`glBlendEquation(mode)` : 최종 src와 최종 dest를 어떻게 합칠 것인가에 대한 결정 함수. (default = GL_FUNC_ADD : 더하기)
#### BackFaceCulling
```cpp
glCullFace(GL_FRONT);
glEnable(GL_CULL_FACE);
```
#### Lighting(Phong reflection model)
###### Ambient
기본 색
###### Diffuse
밝기
###### Specular
빛 반사