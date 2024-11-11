#### SDL
- `SDL_CreateWindow()`
- `SDL_CreateRenderer()`
- `SDL_SetRenderDrawColor()`
- `SDL_RenderClear()`
- `SDL_RenderPresent()`
- `SDL_CreateTexture()`
- `SDL_UpdateTexture()`
- `SDL_RenderCopy()` : 텍스쳐의 특정 부분을 Frame buffer에 복사한다.
- `SDL_Delay()` : 해당 시간 동안 프로세스를 os 스케쥴링에서 제거한다.
- 이벤트 처리
```cpp
void process_input() {
	SDL_Event evn;
	SDL_PollEvent(&evn);
	switch (evn.type) {
		// 이벤트 처리
	}
}
```
- 디스플레이 정보 가져오기
```cpp
SDL_DisplayMode dis_mod;
SDL_GetCurrentDisplayMode(0, &dis_mod);
window_width = dis_mod.w;
window_height = dis_mod.h;
```
- `SDL_SetWindowFullscreen()`
#### GameLoop
- framework
```cpp
int main(void) {
	setup();
	while(true) {
		process_input();
		update();
		render();
	}
	return 0;
}
```
- frame rate 제한
```cpp
int wait_time = previous_frame_time + FRAME_TARGET_TIME - SDL_GetTicks();
if(wait_time > 0) {
	SDL_Delay(wait_time);
}
previous_frame_time = SDL_GetTicks();

//Do engine work
```
#### Color Buffer
- 화면에 그릴 버퍼
- 32Bit 크기로 구성된 배열
- ARGB형식
- 최종적으로 SDL_Texture에 집어 넣고 렌더링
```cpp
void render_color_buffer() {
	SDL_UpdateTexture(
		color_buffer_texture,
		NULL,
		color_buffer,
		sizeof(uint32_t) * window_width
	);
	
	SDL_RenderCopy(
		renderer,
		color_buffer_texture,
		NULL,
		NULL
	);
}
```
#### Projection Matrix
##### perspective projection
![img](https://www.scratchapixel.com/images/perspective-matrix/projectionOpenGL.png?)
[참고 영상1](https://www.youtube.com/watch?v=eoXn6nwV694&t=1397s)
[참고 영상2](https://www.youtube.com/watch?v=md3jFANT3UM&t=381s)
###### 역할
1. 화면 비율(aspect ration : h / w)에 따른 x, y값 조정
	$x * aspect$
2. field of view에 따른 x, y값 조정
	$x * 1 / tan(a)$
	$y * 1 / tan(a)$
3. x, y을 -1~1 사이로 z값을 0 ~ 1 사이로 normalization 하기.
	$(zfar / (zfar - znear) * z) - (afar / (zfar - znear) * znear)$
###### 코드
```cpp
mat4_t mat4_make_perspective(float fov, float aspect, float znear, float zfar) {
mat4_t ret = {{
	{0, 0, 0, 0},
	{0, 0, 0, 0},
	{0, 0, 0, 0},
	{0, 0, 0, 0}
}};

float fov_factor = 1 / tan(fov / 2.0);
float lamda = zfar / (zfar - znear);
ret.m[0][0] = aspect * fov_factor;
ret.m[1][1] = fov_factor;
ret.m[2][2] = lamda;
ret.m[2][3] = -lamda * znear;
ret.m[3][2] = 1.0;

return ret;
}
```
#### View/Camera Matrix
camera position과 반대로 평행이동한다. (camera가 왼쪽으로 이동하면 세상은 오른쪽으로 이동해야하고 오른쪽으로 이동하면 왼쪽으로 이동해야 한다.)
카메라 축에 inversed한 회전을 적용한다. (camera가 위를 향하면 세상은 아래로 회전해야 하고 아래를 향하면 위로 회전해야 한다.)

![img|500](https://europe1.discourse-cdn.com/unity/original/4X/c/9/2/c9224bdebc5a2a3b38d3cecdc626efeb184b89cc.png)

![img](https://docs.tizen.org/application/native/guides/graphics/media/view_matrix.png)

```c
	mat4_t mat4_look_at(vec3_t eye, vec3_t target, vec3_t up) {
		vec3_t z = vec3_sub(target, eye);
		vec3_normalize(&z);
		vec3_t x = vec3_cross(up, z);
		vec3_normalize(&x);
		vec3_t y = vec3_cross(z, x);
	
		mat4_t ret = {{
			{ x.x, x.y, x.z, -vec3_dot(x, eye) },
			{ y.x, y.y, y.z, -vec3_dot(y, eye) },
			{ z.x, z.y, z.z, -vec3_dot(z, eye) },
			{ 0, 0, 0, 1 }
		}};
		return ret;
	}
	```
#### Coordinate System
- 왼손 좌표계 사용 (z가 커지는 방향이 스크린 쪽으로 들어가는 방향)
- model space -> world space(world matrix 곱하기) -> view/camera space(view matrix 곱하기) -> screen space(projection matrix 곱하기)
#### face_t
- 메쉬를 구성하는 삼각형 하나의 버텍스 인덱스를 담고 있는 struct
- clockwise 방향이 앞쪽 방향
#### triangle_t
- 3개의 projection 된 점의 위치를 갖고 있는 struct
#### mesh_t
- `vec3_t*` : 매쉬의 정점들
- `face_t*` : 매쉬의 face들
- transformation 데이터
#### Load .obj file
```cpp
void load_obj_file(const char* filename) {
	FILE* fp;
	fp = fopen(filename, "r");
	if(!fp) return;
	
	char line[256];
	while(fgets(line, sizeof(line), fp)) {
		int len_line = strlen(line);
		if(len_line < 2) continue;
		if(line[0] == 'v' && line[1] == ' ') {
			// vertex information
			vec3_t vertex;
			sscanf(&line[2], "%f %f %f", &vertex.x, &vertex.y, &vertex.z);
			array_push(mesh.vertices, vertex);
		} else if(line[0] == 'f' && line[1] == ' ') {
			// face information
			char buff[3][64];
			sscanf(&line[2], "%s %s %s", buff[0], buff[1], buff[2]);
			face_t face;
			sscanf(buff[0], "%d", &face.a);
			sscanf(buff[1], "%d", &face.b);
			sscanf(buff[2], "%d", &face.c);
			array_push(mesh.faces, face);
		}
	}
	fclose(fp);
}
```
#### Line draw
```cpp
void draw_line(int x0, int y0, int x1, int y1, uint32_t color) {
	// draw line with DDA algorithm
	int delta_x = x1 - x0;
	int delta_y = y1 - y0;
	
	const int longest_len = abs(delta_x) >= abs(delta_y) ? abs(delta_x) : abs(delta_y);
	
	const float x_inc = delta_x / (float)longest_len;
	const float y_inc = delta_y / (float)longest_len;
	
	float cur_x = x0, cur_y = y0;
	for(int i = 0; i < longest_len; i++) {
		draw_pixel(round(cur_x), round(cur_y), color);
		cur_x += x_inc;
		cur_y += y_inc;
	}
}
```
#### Draw triangle
![img](http://www.sunshine2k.de/coding/java/TriangleRasterization/generalTriangle.png)
boundary를 기준으로 위의 삼각형(flat_bottom), 아래 삼각형(flat_top)을 나누어서 그린다.
위의 사진은 v4 포인트의 x값을 찾기 위한 공식이다. (y4의 경우 y2와 같으므로 쉽게 구할 수 있다.)
![img](http://www.sunshine2k.de/coding/java/TriangleRasterization/bresenhamIdea.png)
1, 2의 기울기를 각각 구하고 start_x와 end_x를 구하여 해당 사이의 모든 픽셀에 색을 칠한다.
```cpp
	void draw_flat_bottom_triangle(int x0, int y0, int x1, int y1, int x2, int y2, uint32_t color) {
		// 기울기를 y값 증가에 따른 x값으로 구함
		float inv_slop_1 = (x1 - x0) / (float)(y1 - y0);
		float inv_slop_2 = (x2 - x0) / (float)(y2 - y0);
		
		float start_x = x0, end_x = x0;
		for(int i = y0; i <= y2; i++) {
			draw_line(start_x, i, end_x, i, color);\
			start_x += inv_slop_1;
			end_x += inv_slop_2;
		}
	}
	
	void draw_flat_top_triangle(int x0, int y0, int x1, int y1, int x2, int y2, uint32_t color) {
		// 기울기를 y값 증가에 따른 x값으로 구함
		float inv_slop_1 = (x2 - x0) / (float)(y2 - y0);
		float inv_slop_2 = (x2 - x1) / (float)(y2 - y1);
		
		float start_x = x2, end_x = x2;
		for(int i = y2; i >= y1; i--) 
			draw_line(start_x, i, end_x, i, color);
			start_x -= inv_slop_1;
			end_x -= inv_slop_2;
		}
	}
	
	void draw_filled_triangle(int x0, int y0, int x1, int y1, int x2, int y2, uint32_t color) {
		// y값이 큰 순서대로 정렬
		if(y0 > y1) {
			int_swap(&x0, &x1);
			int_swap(&y0, &y1);
		}
	
		if(y1 > y2) {
			int_swap(&x1, &x2);
			int_swap(&y1, &y2);
		}
	
		if(y0 > y1) {
			int_swap(&x0, &x1);
			int_swap(&y0, &y1);
		}
	
		if(y1 == y2) {
			// 위쪽 삼각형만 그리기
			draw_flat_bottom_triangle(x0, y0, x1, y1, x2, y2, color);
		} else if(y0 == y1) {
			// 아래쪽 삼각형만 그리기
			draw_flat_top_triangle(x0, y0, x1, y1, x2, y2, color);
		} else {
			// midpoint 찾기
			int my = y1;
			int mx = ((x2 - x0) * (y1 - y0)) / (float)(y2 - y0) + x0;
			
			// flat_bottom 삼각형 그리기
			draw_flat_bottom_triangle(x0, y0, x1, y1, mx, my, color);
			// flat_top 삼각형 그리기
			draw_flat_top_triangle(x1, y1, mx, my, x2, y2, color);
		}
	}
	```
#### Back face culling
```cpp
// 삼각형 a, b, c에 대해
// a -> b 벡터와 a -> c벡터를 구한다.
// ab벡터와 ac벡터의 외적을 구하여 삼각형 표면에 수직인 normal 벡터를 구한다.
// a -> camera 벡터를 구한다.
// a -> camera와 normal 벡터를 내적하여 0보다 작을 경우 해당 삼각형은 무시한다.
vec3_t ab = vec3_sub(transformed_vertices[1], transformed_vertices[0]);
vec3_t ac = vec3_sub(transformed_vertices[2], transformed_vertices[0]);
vec3_t normal = vec3_cross(ab, ac);
vec3_t to_camera = vec3_sub(camera_position, transformed_vertices[0]);
if(vec3_dot(normal, to_camera) < 0) {
	continue;
}
```
#### Z Buffer
**1차 해결**
Painter algorithm
![img](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSlxn3IpeyWGH2gbm2odLkma1KZqTruXBKkew&s)
```cpp
	// 삼각형 세 점의 평균 z값을 구한다.
	float avg_depth = (transformed_vertices[0].z + transformed_vertices[1].z + transformed_vertices[2].z) / 3;
	
	// 평균 z 값이 작은 순서대로 정렬
	// z 값이 작을 수 록 마지막에 그린다.
	int len_triangle = array_length(triangles_to_render);
	for(int i = len_triangle - 1; i > 0; i--) {
		for(int j = 0; j < i; j++) {
			if(triangles_to_render[j].avg_depth < triangles_to_render[j + 1].avg_depth) {
				triangle_t tmp = triangles_to_render[j];
				triangles_to_render[j] = triangles_to_render[j + 1];
				triangles_to_render[j + 1] = tmp;
			}
		}
	}
	
	// 정렬된 순서대로 삼각형을 그린다
	int len_triangles = array_length(triangles_to_render);
	for(int i = 0; i < len_triangles; i++) {
		// draw
	}
	```

z값의 평균을 사용하기 때문에 정확하지 않음 (아래와 같은 그림은 표현하지 못한다.)
![img](https://upload.wikimedia.org/wikipedia/commons/thumb/7/78/Painters_problem.svg/220px-Painters_problem.svg.png)
2차 해결
	depth buffer : 픽셀 마다 z값을 저장한 배열
	배열 clear 시 전부 1로 초기화
	픽셀이 그려질때마다 depth buffer에 저장되어 있는 값과 비교하여 더 작으면 갱신하고 그린다.
	```c
	// w값은 projection 되기 전의 z값임
	// 역전된 w이므로 w가 클수록 최종값은 작다
	// 그러므로 1.0에서 역전된 w를 빼줘서 값을 flip한다. (w의 max값은 1 이므로)
	// cf) w값을 역전한 이유는 w값은 linear한 값이 아니기 때문이다.
	inv_w = 1.0 - inv_w;
	if(inv_w < z_buffer[window_width * y + x]) {
		// draw this pixel!
		z_buffer[window_width * y + x] = inv_w;
	}
	```
#### Light and Shading
- Flat Shading
	```cpp
	uint32_t face_color = mesh_face.color;
	// 글로벌 라이트와 법선 벡터를 내적하여 나온 값을 light factor로 사용
	float light_factor = -vec3_dot(global_light.direction, normal);
	// 적용
	face_color = apply_light_intensity(face_color, light_factor);
	```
#### Texturing
- uv 좌표계
	![img|500](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRjGA619DubbMIueWY2zF6_XTKpl-s5tf58-A&s)
- texture란
	`uint32_t`(ARGB)의 배열
- 이론 및 구현
	barycentric coordinates
		![img](https://www.scratchapixel.com/images/ray-triangle/barycentric.png?)
		a의 weight 값은 w이다
		b의 weight 값은 u이다
		c의 weight 값은 v이다
		$w + v + u = 1$이 성립한다
		ABC = ABP + BCP + CAP
		$w = (PC cross PB) / (AB cross AC)$
	```c++
	vec3_t barycentric_weights(vec2_t a, vec2_t b, vec2_t c, vec2_t p) {
		vec2_t ac = vec2_sub(c, a);
		vec2_t ab = vec2_sub(b, a);
		vec2_t ap = vec2_sub(p, a);
		vec2_t pc = vec2_sub(c, p);
		vec2_t pb = vec2_sub(b, p);
		
		// ac x ab 전체 삼각형 크기. 아래는 2d의 외적을 표현. z값의 크기가 나옴
		float total_area = ac.x * ab.y - ac.y * ab.x;
		vec3_t ret = {
			.x = (pc.x * pb.y - pc.y * pb.x) / total_area,
			.y = (ac.x * ap.y - ac.y * ap.x) / total_area
		};
		ret.z = 1.0 - ret.x - ret.y;
		
		return ret;
	}
	```
#### Camera
- FPS camera model
	```cpp
	typedef struct {
		vec3_t position;
		vec3_t direction;
		vec3_t forward_velocity;
		float yaw;	
	} camera_t;
	```
	view matrix 구하기
	```cpp
	vec3_t up = {0, 1, 0};
	vec3_t forward = {0, 0, 1};
	camera.direction = vec3_rotate_y(forward, camera.yaw);
	vec3_t target = vec3_add(camera.position, camera.direction);
	view_mat = mat4_look_at(camera.position, target, up);
	```
#### Clipping
- 이론 
	![img](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSZUmxgiU5V7lDW18VRtOCaxyCTwQnFnzohhw&s)
	clipping이란 위 그림처럼 특정부분에서 벗어난 부분을 잘라 새로운 polygon 형태를 만드는 기술이다
	주의할 점은 여러군데가 잘릴 수 있다는 것
	![img](https://learnwebgl.brown37.net/_images/viewing_frustum.png)
	view frustrum의 경우 6개의 해당하는 clipping plane을 모두 계산한다.
	6개를 계산할 때 first clipping의 결과로 나온 polygon을 그대로 second clipping에 전달해야 한다. (독립적으로 시행 x)
- Plane
	3DPlane은 아래 코드와 같이 점과 방향으로만 정의한다. (점의 경우 plane안에 있는 어떤 점이든 가능하다 즉 3DPlane은 크기가 무한하다.) : [view frustrum의 plane 구하기](https://courses.pikuma.com/courses/take/learn-computer-graphics-programming/lessons/15960054-defining-frustum-planes-points-normals)
	```c
	typedef struct {
		vec3_t point;
		vec3_t normal; // inside 방향
	} plane_t;
 	```
 	plane 안쪽에 존재하는지 구하는 법 : $(Q - Point) (dot) Normal > 0$
 	plane과 2점 사이의 선의 교차 점 구하는 법 : [intersection between line and plane](https://courses.pikuma.com/courses/take/learn-computer-graphics-programming/lessons/16098011-intersection-between-line-plane)
	plane과 polygon의 교차점을 구하는 방법과 plane안쪽의 모든 점을 구하는 방법 : [clipping a polygon against a plane](https://courses.pikuma.com/courses/take/learn-computer-graphics-programming/lessons/15961052-clipping-a-polygon-against-a-plane)
- 코드
	```c
	void clip_polygon_against_plane(polygon_t* polygon, int plane) {
		vec3_t plane_point = frustum_planes[plane].point;
		vec3_t plane_normal = frustum_planes[plane].normal;
		
		vec3_t inside_vertex[MAX_NUM_POLY_VERTICES];
		int num_inside_vertex = 0;
		
		vec3_t* prev_vertex = &polygon->vertices[polygon->num_vertices - 1];
		float prev_dot = vec3_dot(vec3_sub(*prev_vertex, plane_point), plane_normal);  
		
		for(int i = 0; i < polygon->num_vertices; i++) {
			vec3_t* cur_vertex = &polygon->vertices[i];
			float cur_dot = vec3_dot(vec3_sub(*cur_vertex, plane_point), plane_normal);
		
			// 평면을 기준으로 두 점이 다른 쪽에 위치하는 경우 교차점 계산
			if(cur_dot * prev_dot < 0) {
				// linear fomula를 이용하여 교차 점 구하기
				float t = cur_dot / (cur_dot - prev_dot);
				vec3_t insertion_vertex = vec3_add(*cur_vertex, vec3_mul(vec3_sub(*prev_vertex, *cur_vertex), t));
				inside_vertex[num_inside_vertex++] = insertion_vertex;
			}

			// 현재 점이 평면의 안쪽에 있으면 추가
			if(cur_dot > 0) {
				inside_vertex[num_inside_vertex++] = *cur_vertex;
			}
			
			prev_vertex = cur_vertex;
			prev_dot = cur_dot;
		}
		
		// 결과물 복사
		for(int i = 0; i < num_inside_vertex; i++) {
			polygon->vertices[i] = inside_vertex[i];
		}
		polygon->num_vertices = num_inside_vertex;	
	}	  
		
	void clip_polygon(polygon_t* polygon) {
		clip_polygon_against_plane(polygon, LEFT_FRUSTUM_PLANE);
		clip_polygon_against_plane(polygon, RIGHT_FRUSTUM_PLANE);
		clip_polygon_against_plane(polygon, TOP_FRUSTUM_PLANE);
		clip_polygon_against_plane(polygon, BOTTOM_FRUSTUM_PLANE);
		clip_polygon_against_plane(polygon, NEAR_FRUSTUM_PLANE);
		clip_polygon_against_plane(polygon, FAR_FRUSTUM_PLANE);
	}
	```