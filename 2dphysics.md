#### Particle
- 정의
	물리적인 힘을 받는 주체
- 뉴턴 법칙
	Inertia (관성)
		힘이 없으면 움직지 않는다
		힘이 없으면 속도가 변화하지 않는다
	Force, Mass, Acceleration
		힘은 $F=ma$ 법칙을 따른다
	Action and Reaction (작용 반작용)
		물체에 힘을 주면 물체도 반대 방향의 같은 크기로 힘을 준다.
#### Intergration
###### 가속도를 적용하지 않는 경우
Constatant velocity : $p = v * t$ (linear movement)
###### 가속도를 적용하는 경우
![img|600](https://i.pinimg.com/originals/3f/28/da/3f28da2cca77a09b6ebaac7d626620b8.png)
```cpp
// Eluer intergration
particle->velocity += particle->acceleration * deltaTime;
particle->position += particle->velocity * deltaTime;
```
###### 참고 영상
Integration & Movement Simulation : [ref](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28516205-integration-movement-simulation)
Intergration techniques : [ref](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28516496-different-integration-methods)
#### Forces
###### intergrate 적용
1. 최종적인 힘을 구한다.
	`windForce + gravityForce + ... = finalForce`
2. 가속도를 구한다. 
	$F = ma$ 에 의하여 $a = F / m$
```cpp
void Particle::Integrate(float dt) {
	acceleration = sumForces / mass;
	
	// Euler integration
	velocity += acceleration * dt;
	position += velocity * dt;
	
	ClearForces();
}
```
#### Weight force
실제로는 무게에 따라 떨어지는 속도가 다르지 않다.
mass는 절대 변하지 않는 값.
weight는 중력에 따라 달라지는 값.
###### 공식
$w = mass * gravity$
###### 코드
```cpp
Vec2 weightForce = Vec2(0.f, particle->mass * gravity);
//...
acceleration = sumForces / mass; // particle->mass를 곱한 값은 invert된다.
```
#### Drag force
resistive force(항력)
빠르게 움직일 수록 커진다.
###### 공식
항력 (Fd) = 0.5 * 항력 계수 (Cd) * 유체 밀도 (ρ) * 단면적 (A) * 속도^2 (v^2) 
- **항력 계수 (Cd):** 물체의 모양에 따라 달라지는 값으로, 실험이나 시뮬레이션을 통해 구할 수 있습니다.
- **유체 밀도 (ρ):** 유체의 질량을 부피로 나눈 값입니다.
- **단면적 (A):** 유체의 흐름에 수직인 물체의 단면적입니다.
- **속도 (v):** 물체의 속도입니다.
###### 코드
항력 계수, 유체 밀도, 단면적은 유연한 const값으로 대체하여 복잡한 계산을 최소화.
```cpp
Vec2 Force::GenerateDragForce(Particle& particle, float k) {
	Vec2 ret = Vec2(0, 0);
	if(particle.velocity.MagnitudeSquared() > 0) {
		Vec2 dir = particle.velocity.Normal() * -1.0;
		float factor = k * particle.velocity.MagnitudeSquared();
		ret = dir * factor;
	}
	return ret;
}
```
#### Friction force
kinetic friction : 물체가 포면을 따라 움직일때 발생하는 velocity의 반대 힘 
static friction : 물체가 정지하게 유지 시키는 힘 velocity의 반대 힘
###### 공식
**Ff = μ * FN**
- **Ff**: 마찰력 (friction force) - 물체의 움직임을 방해하는 힘
- **μ**: 마찰 계수 (coefficient of friction) - 두 표면의 거칠기 정도를 나타내는 값 (단위 없음)
- **FN**: 수직 항력 (normal force) - 접촉면이 물체를 밀어 올리는 힘
###### 코드
마찰 계수, 수직 항력은 유연한 const값으로 대체하여 복잡한 계산을 최소화.
```cpp
Vec2 Force::GenerateFrictionForce(Particle& particle, float k) {
	Vec2 ret = Vec2(0, 0);
	float velocityMag = particle.velocity.MagnitudeSquared();
	Vec2 dir = particle.velocity.UnitVector() * -1.0;
	float frictionMag = k * velocityMag;
	ret = dir * frictionMag;
	return ret;
}
```
#### Gravitational attraction
만유 인력
###### 공식
**F = G _(m1_ m2) / r²**
- **F**: 두 물체 사이의 만유인력 (gravitational force)
- **G**: 만유인력 상수 (gravitational constant) - 6.674 × 10⁻¹¹ N⋅m²/kg²
- **m1**: 첫 번째 물체의 질량 (mass)
- **m2**: 두 번째 물체의 질량 (mass)
- **r**: 두 물체의 질량 중심 사이의 거리 (distance)
###### 코드
```cpp
// a에서 b로 향하는 만유 인력 구하기
Vec2 Force::GenerateGravitationalForce(const Particle& a, const Particle& b, float G) {
	Vec2 ab = b.position - a.position;
	float dist = ab.MagnitudeSquared();
	Vec2 attractionDir = ab.UnitVector();
	float attractionMag = G * (a.mass * b.mass / dist);
	return attractionDir * attractionMag;
}
```
#### Spring force
###### 공식
**F = -kx**
- **F**: 스프링이 가하는 힘 (spring force) - 복원력이라고도 불리며, 스프링의 평형 위치로 돌아가려는 방향으로 작용합니다.
- **k**: 스프링 상수 (spring constant) - 스프링의 강성을 나타내는 값으로, 스프링을 단위 길이만큼 늘리거나 압축하는 데 필요한 힘의 크기를 나타냅니다. 단위는 N/m 입니다.
- **x**: 스프링의 변위 (displacement) - 스프링이 평형 위치에서 늘어나거나 압축된 길이를 나타냅니다. 단위는 미터(m)입니다.
###### 코드
```cpp
Vec2 Force::GenerateSpringForce(const Body& particle, Vec2 anchor, float restLength, float k) {
	Vec2 d = particle.position - anchor;
	float delta = d.Magnitude() - restLength;
	float springMag = -k * delta;
	return d.UnitVector() * springMag;
}
```
#### Rigid body
형체가 변하지 않는 물체.
center of mass(물체의 중점)이 존재한다.
###### 코드
```cpp
enum ShapeType {
	BOX, POLYGON, CIRCLE
};

struct Shape {

};

struct CircleShape : public Shape {}
struct PolygonShape : public Shape {}
// ...

// 기존 Particle struct를 Body로 교체
struct Body {
	//...
	Shape* shape;
}
```
#### Angular velocity and Acceration
Torque and Moment of inertia
Torque 정의
물체를 **회전시키려는 힘**

공식
**τ = r × F = rFsin(θ)**
- **τ (tau):** 토크 (벡터량, 단위: N⋅m 또는 lb⋅ft)
- **r:** 회전축에서 힘의 작용점까지의 거리 벡터 (벡터량)
- **F:** 작용하는 힘 벡터 (벡터량)
- **θ:** r과 F 사이의 각도
![img|600](https://cdn1.byjus.com/wp-content/uploads/2023/03/Introduction-to-Torque-and-Its-Applications-2.png)

실제 게임에서 사용하는 방법은 실시간성과 성능을 위해 토크를 직접적으로 적용하기보다는 관성 모멘트(Moment of Inertia)와 각가속도(Angular Acceleration)를 사용하여 회전 운동을 계산

**1. 관성 모멘트 (Moment of Inertia, I)**

- 회전 운동에서 물체가 회전 상태를 유지하려는 정도를 나타내는 물리량.
- 관성 모멘트가 클수록 회전 상태를 변화시키기 어렵다.

**2. 각가속도 (Angular Acceleration, α)**

- 시간에 따른 각속도의 변화율을 나타내는 물리량.

따라서 아래와 같은 공식을 사용

**τ = Iα**
- **τ (tau):** 물체에 작용하는 토크 (Torque), 단위: N⋅m
- **I:** 물체의 관성 모멘트 (Moment of Inertia), 단위: kg⋅m²
- **α (alpha):** 물체의 각가속도 (Angular Acceleration), 단위: rad/s²

**이 공식은 뉴턴의 제2 운동 법칙 (F = ma)의 회전 운동 버전이라고 할 수 있음.**

추가적으로 각가속도의 경우
**α = dω / dt**
- **dω (omega):** 각속도 (Angular Velocity)의 변화량, 단위: rad/s
- **dt** : 시간의 변화량

cf) 각 모양에 따른 moment of inertia : [ref](https://en.wikipedia.org/wiki/List_of_moments_of_inertia)

코드
```cpp
struct Body {
	// ...

	// 회전에 관련된 힘
	float rotation;
	float angularVelocity;
	float angularAcceleration;
	float invI;
	float sumOfAngularForce;

	void IntegrateAngular(float dt) {
		angularAcceleration = sumOfAngularForce * invI; 
		angularVelocity += angularAcceleration * dt;
		rotation += angularVelocity * dt;
	}
};
```
#### Collision
##### Collision detect
###### circle vs circle
두 원의 반지름의 합이 현재 두 원 사이의 거리 보다 크면 충돌
```cpp
bool CollisionDetection::IsCollidingCircleCircle(Body* a, Body* b) {
	CircleShape* aCircle = static_cast<CircleShape*>(a->shape);
	CircleShape* bCircle = static_cast<CircleShape*>(b->shape);

	const float radiusSum = aCircle->radius + bCircle->radius;
	const float sqrtDist = (a->position - b->position).MagnitudeSquared();

	return sqrtDist <= (radiusSum * radiusSum);
}
```
###### box vs box
simple solve
AABB
```cpp
bool AABB(int al, int ar, int at, int ab, int bl, int br, int bt, int bb) {
	return (ar > bl && al < br) && (ab < bt && at > bb);
}
```
###### polygon vs polygon
SAT
두 도형을 완변히 두 영역으로 나누는 선이 존재하는지 여부 파악을 이용한 알고리즘
오직 convex polygon(폴리곤 내부에 두점을 두고 이었을 때 선이 항상 폴리곤 내부에 존재하는 형태)에서만 적용 가능함
![img](https://dyn4j.org/assets/posts/2010-01-01-sat-separating-axis-theorem/sat-ex-3.png)
cf) [참고 영상](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28541738-sat-separating-axis-theorem)

코드
```cpp
float PolygonShape::FindMinimumSeperation(const PolygonShape* other) const {
	float seperation = std::numeric_limits<float>::lowest();

	for(int i = 0; i < worldVertices.size(); i++) {
		const Vec2& vert = worldVertices[i];
		Vec2 normal = EdgeAt(i).Normal();

		float minSeperation = 0;		
		for(auto& otherVert : other->worldVertices) {
			float projection = normal.Dot(otherVert - vert);
			minSeperation = std::min(minSeperation, projection);
		}
		seperation = std::max(seperation, minSeperation);
	}

	return seperation;
}

bool CollisionDetection::IsCollidingPolygonPoloygon(Body* a, Body* b, Contact& contact) {
	// SAT algorithm
	const PolygonShape* aPoly = static_cast<PolygonShape*>(a->shape);
	const PolygonShape* bPoly = static_cast<PolygonShape*>(b->shape);

	float abSeperation = aPoly->FindMinimumSeperation(bPoly, aAxis, aVertex);
	if(abSeperation >= 0) {
		return false;
	}

	float baSeperation = bPoly->FindMinimumSeperation(aPoly, bAxis, bVertex);
	if(baSeperation >= 0) {
		return false;
	}

	return true;
}
```
![img|600](https://files.cdn.thinkific.com/file_uploads/167815/images/800/241/d73/MinAxisSeparation.gif)
cf) [참고 영상](https://www.youtube.com/watch?v=-EsWKT7Doww)
###### Circle vs Polygon
1. 원과 가장 가까운 폴리곤의 선을 찾음
2. 두가지의 Case
	Case 1. 원의 중점이 안에 있는 경우
		무조건 충돌이므로 `true`반환
	Case 2. 원의 중점이 밖에 있는 경우
		Case 1. a 구역(가까운 선의 왼쪽 구역)
		Case 2. b 구역(가까운 선의 중앙 구역)
		Case 3. c 구역(가까운 선의 오른쪽 구역)
```cpp
bool CollisionDetection::IsCollidingCirclePolygon(Body* a, Body* b, Contact& contact) {
    const CircleShape* circle = static_cast<CircleShape*>(a->shape);
    const PolygonShape* polygon = static_cast<PolygonShape*>(b->shape);

    float distToEdge = std::numeric_limits<float>::lowest();
    Vec2 closestEdgeStart;
    Vec2 closestEdgeEnd;

    for(int i = 0; i < polygon->worldVertices.size(); i++) {
        Vec2 edge = polygon->EdgeAt(i);
        Vec2 edgeNormal = edge.Normal();
        Vec2 toCircle = a->position - polygon->worldVertices[i];

        float projection = toCircle.Dot(edgeNormal);

        if(projection > distToEdge) {
            distToEdge = projection;
            closestEdgeStart = polygon->worldVertices[i];
            closestEdgeEnd = closestEdgeStart + edge;
        }
    }

    if(distToEdge > 0.f) {
        // circle centr is outside of the polygon
        Vec2 v1 = a->position - closestEdgeStart;
        Vec2 v2 = closestEdgeEnd - closestEdgeStart;

        if(v1.Dot(v2) < 0) {
            float distToCircle = v1.Magnitude();
            if(distToCircle < circle->radius) {
		        // a region
                return true;
            }
        } else {
            v1 = a->position - closestEdgeEnd;
            v2 = closestEdgeStart - closestEdgeEnd;

            if(v1.Dot(v2) < 0) {
                float distToCircle = v1.Magnitude();
                if(distToCircle < circle->radius) {
	                // b region
                    return true;
                }               
            } else {
                if(distToEdge < circle->radius) {
	                // c region
                    return true;
                }
            }
        }
    } else {
        // circle center is inside the polygon
        // therefore always colliding
        contact.a = b;
        contact.b = a;
        contact.depth = circle->radius - distToEdge;
        contact.normal = (closestEdgeEnd - closestEdgeStart).Normal();
        contact.start = a->position - contact.normal * circle->radius;
        contact.end = contact.start + contact.normal * circle->radius;

        return true;
    }

    return false;
}
```
cf) [참고 영상](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28548274-exercise-circle-polygon-edge-regions)
##### Collision resolve
###### contact
```cpp
// a와 b의 충돌 정보 (모든 데이터는 a를 기준으로)
struct Contact {
	Body* a;
	Body* b;

	Vec2 start, end;
	Vec2 normal;
	float depth;

	Contact() = default;
	~Contact() = default;
};
```
참고 영상) [ref](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28540127-collision-contact-information)
###### circle collision contact
```cpp
contact.a = a;
contact.b = b;
contact.normal = ab.UnitVector();
contact.start = b->position + (-contact.normal * bCircle->radius);
contact.end = a->position + (contact.normal * aCircle->radius);
contact.depth = (contact.end - contact.start).Magnitude();
```
참고 영상) [17:50초](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28540223-collision-information-code)
###### polygon collision contact
```cpp
contact.a = a;
contact.b = b;

if(abSeperation > baSeperation) {
	contact.depth = -abSeperation;
	contact.normal = aAxis.Normal();
	contact.start = aVertex;
	contact.end = aVertex + contact.normal * contact.depth;
} else {
	contact.depth = -baSeperation;
	contact.normal = -bAxis.Normal();
	contact.start = bVertex - contact.normal * contact.depth;
	contact.end = bVertex;
}
```
###### polygon and circle collision contact
```cpp
// ================================
// region a
// ================================
Vec2 v1 = a->position - closestEdgeStart;
Vec2 v2 = closestEdgeEnd - closestEdgeStart;

float distToCircle = v1.Magnitude();
contact.a = b;
contact.b = a;
contact.depth = circle->radius - distToCircle;
contact.normal = v1 / distToCircle;
contact.start = a->position - contact.normal * circle->radius;
contact.end = contact.start + contact.normal * contact.depth;

// ================================
// region b
// ================================
v1 = a->position - closestEdgeEnd;
v2 = closestEdgeStart - closestEdgeEnd;

distToCircle = v1.Magnitude();
contact.a = b;
contact.b = a;
contact.depth = circle->radius - distToCircle;
contact.normal = v1 / distToCircle;
contact.start = a->position - contact.normal * circle->radius;
contact.end = contact.start + contact.normal * contact.depth;

// ================================
// region c
// ================================
contact.a = b;
contact.b = a;
contact.depth = circle->radius - distToEdge;
contact.normal = (closestEdgeEnd - closestEdgeStart).Normal();
contact.start = a->position - contact.normal * circle->radius;
contact.end = contact.start + contact.normal * contact.depth;

// ================================
// 원의 중점이 폴리곤 내부에 존재
// 폴리곤에 가장 가까운 선으로 밀어내기 정보 저장
// ================================
contact.a = b;
contact.b = a;
contact.depth = circle->radius - distToEdge;
contact.normal = (closestEdgeEnd - closestEdgeStart).Normal();
contact.start = a->position - contact.normal * circle->radius;
contact.end = contact.start + contact.normal * circle->radius;
```
##### Resolve
###### Projection method
충돌한 물체의 위치 수정

얼마만큼 이동해야 하는가?
질량이 큰 오브젝트는 적게 움직이고 질량이 작은 오브젝트는 크게 움직이게 해야한다.
$dist = depth * bMass / (aMass + bMass)$의 공식을 사용하지만 실제 구현에서는 invMass만을 저장하고 있기에
$dist = depth / (aInvMass + bInvMass) * aInvMass$의 공식을 적용.

```cpp
void Contact::ResolvePenetration() {
	if(a->IsStatic() && b->IsStatic()) return;

	const float da = depth / (a->invMass + b->invMass) * a->invMass;
	const float db = depth / (a->invMass + b->invMass) * b->invMass;
	
	a->position -= normal * da;
	b->position += normal * db;
}
```

cf) invMass를 사용하는 이유
1. 매번 나눗셈 연산을 피하기 위해 (실제 힘을 적용할때는 mass의 역수만을 필요로 한다 그외에도 앞으로 나올 많은 공식들이 질량의 역수를 활용)
3. static object를 표현하기 위해
	invMass가 0이면 무한 질량을 가지는 걸로 간주하면 **해당하는 모든 연산들이 상쇄된다**
```cpp
bool Body::IsStatic() {
	// float와 0을 비교하는건 위험하다
	// 따라서 현재 값과 0의 차가 무시할 정도로 작으면 0이라고 간주
	// cf) 이와 같은 방법도 완전한 방법은 아님
	const float epsilon = 0.005f;
	return fabs(invMass - 0.0) < epsilon;
}
```
######  Impulse method
충돌한 물체의 속도를 수정(Intergration(힘 -> 가속도 -> 속도 -> 위치)을 적용하지 않고 직접 적용)

Momentum(운동량)
**p = mv**
- **p:** 운동량 (Momentum), 단위: kg⋅m/s
- **m:** 물체의 질량 (Mass), 단위: kg
- **v:** 물체의 속도 (Velocity), 단위: m/s

$mva + mvb = mv'a + mv'b$
두 물체의 운동량의 합은 항상 같다. (두 물체가 충돌하였을때 다른 물체에게 운동량을 전달해주려면 자신의 운동량을 빼서 준다)

Impulse(충격량)
힘과 시간으로써의 공식
**J = FΔt**
- **J:** 충격량 (Impulse), 단위: N⋅s
- **F:** 물체에 작용하는 평균 힘 (Average Force), 단위: N
- **Δt:** 힘이 작용하는 시간 (Time Interval), 단위: s

Momentum(운동량)의 변화량으로써의 공식
**J = Δp = mΔv**
- **J:** 충격량 (Impulse), 단위: N⋅s
- **Δp:** 운동량의 변화 (Change in Momentum), 단위: kg⋅m/s
- **m:** 물체의 질량 (Mass), 단위: kg
- **Δv:** 속도의 변화 (Change in Velocity), 단위: m/s

$mΔv$ 유도 방법
$F=ma$ -> $F = m * Δv / Δt$ -> $F * Δt = m * Δv$ -> $J = FΔt$에 의하여 위와 같은 공식이 유도됨

따라서 $Δv = J / m$이므로 impulse 적용 코드는 아래와 같다
```cpp
void Body::ApplyImpulse(const Vec2& impulse) {
    if(IsStatic()) {
        return;
    }
    velocity += impulse * invMass;
}
```

그렇다면 어느정도의 impulse를 줘야하는가?
linear impulse : [영상](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28541521-deriving-the-linear-impulse-formula)
angular impulse : [영상1](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28546230-linear-angular-velocity-at-point), [영상2](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28546863-post-collision-velocity-at-point), [영상3](https://courses.pikuma.com/courses/take/game-physics-engine-programming/lessons/28546913-computing-linear-angular-impulse)
###### 실제 게임 엔진에서는
현재 구현에서는 매 프레임마다 모든 오브젝트의 충돌 검사를 진행하지만 실제 게임 엔진에서는 코스트가 별로 들지 않는 충돌 검사(circle, box, sphere etc)로 대충 해보고 컬링을 거친뒤에 충돌 가능성이 있는 오브젝트만 디테일하게 충돌 검사를 실시한다.
#### Penetration Constraints
**강체(rigid body)들의 움직임을 제한하는 기술**

jitter 현상(충돌 해결 과정에서 2개의 물체가 서로를 미는데 2개의 물체만 봤을때는 해결법이 옳지만 전체적인 오브젝트를 보았을때는 1-2번이 서로를 밀고 2-3번이 서로를 각자 밈으로 인해 떨리는 현상이 일어난다) 방지
###### Naive한 솔루션
충돌 해결을 100%로 처리하지 말고 적당한 여유분을 두고 해결한다
```cpp
void Contact::ResolvePenetration() {
    if(a->IsStatic() && b->IsStatic()) return;

    const float da = depth / (a->invMass + b->invMass) * a->invMass;
    const float db = depth / (a->invMass + b->invMass) * b->invMass;

	// 80%정도로만 해결한다.
    a->position -= normal * da * 0.8;
    b->position += normal * db * 0.8; 

    a->shape->UpdateVertices(a->position, a->rotation);
    b->shape->UpdateVertices(b->position, b->rotation);
}
```
대신 충돌 해결을 여러번 실행한다
``` cpp
void World::Update(float deltaTime) {
    // ...
	for(int i = 0; i < 10; i++) {
		CheckCollisions();
	}
}
```