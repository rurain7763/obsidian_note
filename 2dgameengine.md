#### SDL
- `IMG_Load()` : 이미지 파일을 읽어드려 `SDL_Surface`로 반환.
#### DeltaTime
- FPS 제한
```cpp
int timeToWait = (_prevFrameMilliSecs + MILLISECS_PER_FRAME) - SDL_GetTicks();
if(timeToWait > 0) {
	SDL_Delay(timeToWait);
}
_prevFrameMilliSecs = SDL_GetTicks();
```
- DeltaTime
```cpp
double deltaTime = (SDL_GetTicks() - _prevFrameMilliSecs) / 1000.0;
```
#### Logger
- 클래스
```cpp
enum LogType {
	LOG_TYPE_INFO,
	LOG_TYPE_ERROR
};

struct LogEntry {
	LogType type;
	std::string message;
};

class Logger {
private:
	static std::vector<LogEntry> _logs;

public:
	static void Log(const std::string& message);
	static void Err(const std::string& message);
};
```
- 사용 예시
```cpp
#define LOG(fmt, ...) \
	{ \
		// 현재 시간을 string 형태로 가져온다
		std::time_t now = std::chrono::system_clock::to_time_t(std::chrono::system_clock::now()); \
		std::string dateStr(30, 0); \
		std::strftime(&dateStr[0], dateStr.size(), "%Y-%m-%d %H:%M:%S", std::localtime(&now)); \
		// 모든 메세지를 합친다
		char buffer[256]; \
		snprintf(buffer, sizeof(buffer), "%s [%s: %d][%s] " fmt, \
		dateStr.c_str(), __FILE__, __LINE__, __func__, ##__VA_ARGS__); \
		Logger::Log(buffer); \
	}
```
#### ECS
###### 1차 접근
![img||300](https://www.codeproject.com/KB/game/gameclassinheritance/Flow.png)
최상위 부모 Class를 Obj로 두고 모두 Obj를 상속받게 한다. 하지만 상속은 아래와 같은 문제점이 있다.
![img|300](https://media.geeksforgeeks.org/wp-content/uploads/20191222084637/Diamond1.png)
다중 상속 시 diamond problem(ex. class2와 class3에서 class1의 함수를 오버라이드 했을때 어떤 함수를 호출해야 하나?) 발생.
###### 2차 접근
Component-Based Design
![img|500](https://cdn.hytale.com/cad0c691a614edc8c7f07aee46c605b8_flecs_diagram_02.jpg)	
1차 구현은 아래와 같다.
```cpp
class Component {
	virtual Update();
}

class TransformComponent {
	virtual void Update() {
		//...
	}
}

class Entity {
	vector<Component*> components;
	void Update();
}	
```
위와 같은 코드의 문제점은 동적 할당된 여기저기의 `Component`를 찾아서 `Update`해야 하므로 CPU에서 매번 로딩해야하는 문제점 발생.

따라서 아래와 같은 방식(ECS)으로 해결.
![img|600](https://img.itch.zone/aW1nLzEzNzMzNDA2LnBuZw==/original/j%2FRGUX.png)
최종 구현은 아래와 같다.
```cpp
class Entity {
	// entity 오직 id값만 가진다
	int id;
}

struct Component {
	// component의 경우 구현부 없어 데이터만 가진다.
	// data
}

class System {
	// 타겟 entity를 가지고 실제 동작을 수행한다.
	void AddEnity(const Entity& entity);
	void RemoveEntity(const Entity& entity);
	const std::vector<Entity>& GetEntities() const;
};
```
###### 대체 외부 라이브러리
EnTT (for C++ projects): [https://github.com/skypjack/entt](https://github.com/skypjack/entt)
Flecs (for C projects): [https://github.com/SanderMertens/flecs](https://github.com/SanderMertens/flecs)
#### Asset Manager
###### 코드
```cpp
class AssetManager {
	void AddTexture(SDL_Renderer* renderer, const std::string& assetID, const std::string& filePath);
	SDL_Texture* GetTexture(const std::string& assetID) const;
	// TODO: ttf, sound 등등 추가...

	void ClearAssets();
	
	std::map<std::string, SDL_Texture*> _textures;
	// TODO: ttf, sound 등등 추가...
};
```
#### Animation System
###### AnimationComponent
```cpp
struct AnimationComponent {
	int numFrames;
	int frameSpeedRate;
	int currentFrame;
	int startTime;
};
```
###### AnimationSystem
```cpp
class AnimationSystem : public System {
public:
	AnimationSystem() {
		RequireComponent<SpriteComponent>();
		RequireComponent<AnimationComponent>();
	}

	void Update() {
		for(auto& entity : GetEntities()) {
			auto& sprite = entity.GetComponent<SpriteComponent>();
			auto& animation = entity.GetComponent<AnimationComponent>();

			// 현재 프레임을 구해서 spritecomponent의 해당하는 srcX로 변경
			animation.currentFrame = ((SDL_GetTicks() - animation.startTime) * animation.frameSpeedRate / 1000) % animation.numFrames;
			sprite.srcRectX = sprite.width * animation.currentFrame;
		}
	}
};
```
#### Collision System
###### BoxColliderComponent
```cpp
struct BoxColliderComponent {
	int width, height;
	glm::vec2 offset;
};
```
###### CollisionSystem
```cpp
class CollisionSystem : public System {
public:
	CollisionSystem() {
		RequireComponent<TransformComponent>();
		RequireComponent<BoxColliderComponent>();
	}
	
	void Update() {
		auto& entities = GetEntities();
		for(int i = 0; i < entities.size(); i++) {
			auto& transformA = entities[i].GetComponent<TransformComponent>();
			auto& boxColliderA = entities[i].GetComponent<BoxColliderComponent>();
			
			// y축이 스크린 좌표계이므로 반전임을 주의!
			float lA = transformA.position.x + boxColliderA.offset.x;
			float rA = lA + boxColliderA.width;
			float tA = transformA.position.y + boxColliderA.offset.y;
			float bA = tA + boxColliderA.height;
			
			for(int j = i + 1; j < entities.size(); j++) {
				auto& transformB = entities[j].GetComponent<TransformComponent>();
				auto& boxColliderB = entities[j].GetComponent<BoxColliderComponent>();
	
				float lB = transformB.position.x + boxColliderB.offset.x;
				float rB = lB + boxColliderB.width;
				float tB = transformB.position.y + boxColliderB.offset.y;
				float bB = tB + boxColliderB.height;
	
				if(AABB(lA, rA, tA, bA, lB, rB, tB, bB)) {
					//TODO: 충돌!
				}
			}
		}
	}
	
	bool AABB(float lA, float rA, float tA, float bA, float lB, float rB, float tB, float bB) const {
		return rA > lB && lA < rB && bA > tB && tA < bB;
	}
};
```
#### Event System
###### 이론
![img](https://denyskryvytskyi.github.io/assets/img/event-system/class-diagram.png)
장점 : 커플링을 없앤다.
###### 구현 종류
**blocking**
이벤트가 발생하면 즉시 처리한다.
```cpp
class DamageSystem {
	void Setup() {
		eventBus->Subscribe<>(); //
	}

	void OnCollision() {
		// ...
	}
};
class CollisionSystem {
	void Update() {
		//...
		evnetBus->EmitEvent<>();
	}
};
```

**unblocking**
발생 시 바로 처리 하지 않고 마지막에 몰아서 처리한다.
```cpp
class DamageSystem {
	void Update() {
		if(eventBus->GetEvnet<>()) {
			// ...
		}
	}
};

class CollisionSystem {
	void Update() {
		// ...
		evnetBus->EmitEvent<>();
	}
};
```
#### Tagging & Groupping Entities
```cpp
class Registry {
	void TagEntity(const Entity& entity, const std::string& tag);
	bool EntityHasTag(const Entity& entity, const std::string& tag) const;
	Entity GetEntityByTag(const std::string& tag) const;
	void RemoveEntityTag(const Entity& entity);
	
	void GroupEntity(const Entity& entity, const std::string& group);
	bool EntityBelongsToGroup(const Entity& entity, const std::string& group) const;
	std::vector<Entity> GetEntitiesByGroup(const std::string& group) const;
	void RemoveEntityGroup(const Entity& entity);

	std::unordered_map<std::string, Entity> _entityPerTag;
	std::unordered_map<Entity, std::string> _tagPerEntities;

	std::unordered_map<std::string, std::set<Entity>> _entitiesPerGroup;
	std::unordered_map<Entity, std::string> _groupPerEntity;
};
```
#### Data-Oriented Design(데이터 지향 설계)
###### OOP(Object-oriented programming)
정의
	프로그램을 수많은 '객체(object)'라는 기본 단위로 나누고 이들의 상호 작용으로 서술하는 방식
DOD와 비교한 장점
	사람이 읽기 쉽다
DOD와 비교한 단점
	데이터가 continuously(연속적)이지 않아 cache miss를 유발한다.
	array of structs(aos) vs strcust of arrays(soa)
	![img|500](https://www.researchgate.net/publication/261860648/figure/fig27/AS:667891238178831@1536249088802/AOS-vs-SOA-data-access-patterns.png)
	cache(cpu에 존재하는 빠르지만 작은 저장 공간, l1, l2, l3, ...으로 나뉘어진다.(l뒤의 숫자가 커질 수록 느린 캐쉬)) miss 발생
	![img|500](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRygKklleBt626TeJ93-9JQII_vxfXyXj0Fag&s)
예시 코드
```cpp
for(uint64_t index = 0; index < game_object_count; index++) {  
    if (oop_game_objects[index].deleted == 1) {  
        continue;  
    } 
    if (oop_game_objects[index].is_visible == 1) {  
        foo = index;  
    }  
}
```
###### DOD
정의
	CPU 캐시의 효율적인 사용에서 동기를 얻은 프로그램 최적화 접근 방식 
	데이터 레이아웃에 초점을 맞추고 필요할 때 필드를 분리 및 정렬
OOP와 비교한 장점
	CPU 캐시의 효율적인 사용으로 특정 상황(특정한 데이터들의 묶음만이 필요한 상황)에서 빠른 속도로 데이터에 접근한다
OOP와 비교한 단점
	사람이 해석하기 불편하다
예시 코드
```cpp
for(uint64_t index = 0; index < game_object_count; index++) {  
    if (dod_game_object_deleted[index] == 1) {  
        continue;  
    }
    
    if (dod_game_object_is_visible[index] == 1) {  
        foo = index;  
    } 
}
```
###### Cache miss debug 라이브러리
valgrind : [https://valgrind.org/]
#### Text rendering
###### TTF(True-Type Font)
###### 그리기 코드
```cpp
TTF_Font* font = assetMg->GetFont(label.assetID);
SDL_Surface* surface = TTF_RenderText_Blended(font, label.text.c_str(), label.color);
SDL_Texture* texture = SDL_CreateTextureFromSurface(renderer, surface);
SDL_FreeSurface(surface);

int textureW = 0, textureH = 0;
SDL_QueryTexture(texture, NULL, NULL, &textureW, &textureH);

SDL_Rect dstRt = {
	.x = static_cast<int>(transform.position.x - (label.isFixed ? 0 : camera.x)),
	.y = static_cast<int>(transform.position.y - (label.isFixed ? 0 : camera.y)),
	.w = static_cast<int>(textureW * transform.scale.x),
	.h = static_cast<int>(textureH * transform.scale.y)
};

SDL_RenderCopy(renderer, texture, NULL, &dstRt);
SDL_DestroyTexture(texture);
```
#### Scripting
###### scripting language
코어 엔진 개발 언어(native language)와는 별개로 게임을 script할 수 있게 지원하는 언어
느리지만 개발하기 편하다
유명 언어 : Lua, wren, c#, python, javascript
###### Lua
designed to be scripting language
not stand-alone language
small, fast, extensible, protable
easily intergrated with c ans c++ by using c api
###### Sol
lua의 c api를 modern c++ 맞게 wrapping한 라이브러리
######  Lua 기본
- Hello world
```cpp
void TestLua() {
sol::state lua;
	lua.open_libraries(sol::lib::base);
	lua.script_file("./assets/scripts/test.lua");

	int someVariable = lua["some_variable"];
	std::cout << someVariable << '\n';

	//if a key exists in the Lua table, we can use a sol::optional or even define a default value with .get_or() function.
}
```

```lua
-- this is lua global variable
some_variable = 7 * 6
user_name = "Lee"

print("hello"..user_name)
```
- call function
```lua
function factorial(n)
	res = 1
	for i = 1, n, 1
	do
		res = res * i
	end
	print(res)
end

factorial(7)
```

```lua
-- 재귀 버전
function factorial(n)
	if n == 1 then
		return 1
	end
	return n * factorial(n - 1)
end
print(factorial(7))
```
- call lua function from c++
```cpp
sol::function factorial = lua["factorial"];
std::cout << "factorial from lua : " << (int)factorial(5) << '\n';
```
- call c++ function from lua
```cpp
int cube(int n) {
	return n * n * n;
}

void TestLua() {
	sol::state lua;
	lua.open_libraries(sol::lib::base);
	lua["cube"] = cube;
}
```

```lua
print("this is from c++ "..cube(3))
```
- loop all element in lua table in c++
```cpp
sol::table table = lua["?"];
int i;
while(true) {
	sol::optional<sol::table> opt = table[i];
	if(opt == sol::nullopt) break;
	// ...
	i++;
}
```
- lua에게 user_type 알려주기
```cpp
lua.new_usertype<Entity>(
	"entity",
	"get_id", &Entity::GetID,
);
``` 
