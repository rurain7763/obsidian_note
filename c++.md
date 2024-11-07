#### 부동 소수
- 소수의 이진수 변환
	부호 (1 bit) + 지수 (8 bit) + 유효숫자 (23 bit)
	ex) -3.375
	1. 3 + 0.375
	2. 3 = 0b11, + 0.375 = ?
	3. 0.375 = 0.5 x 0 + 0.25 x 1 + 0.125 x 1
	4. 0.375 = 0b0.011
	5. 0b11 + 0b0.011 = 0b11.011 -> 0b1.1011 x 2 ^ 1
	6. 부호(1) + 지수(127 + 1) + 유효숫자(1011)
	7. 0b1100 0000 0101 1000 0000 0000 0000 0000 으로 저장됨
- resolution
	표현 가능한 두 수 사이의 최소 간격
	Matisa : 유효숫자 부분
	Exponent : 지수
#### 문자
- char = ascii, wchar_t = unicode
- unicode의 대표적 표현 방식
	1. utf8 : 1btye ~ 3byte
	2. utf16 : 2byte 고정 혹은 예외적 문자는 4byte지만 무시 가능
#### 엔디안
- 리틀 엔디안(현재 가장 많이 사용) : 작은 비트 부터 저장(거꾸로 저장한다고 생각하면 됨)
	1. 장점 : 특정 부분 데이터를 가져오는데 용이하다.
- 빅 엔디안 : 큰 비트 부터 저장(사람이 보기 편하게 저장한다고 보면 됨)
	1. 장점 : 수의 비교에 용이하다.
	
![img|600](https://yoginsavani.com/wp-content/uploads/2023/05/Big-endian-and-Little-endian.png)
#### 데이터 영역
- 스택
- 힙
- 데이터 : .data, .bss, .rodata(읽기 전용)
- 코드 : .text
#### 전역 변수
- data 영역과 bss 영역에 저장됨.
- data 영역은 초기값이 있는 변수이고 bss는 아님
- 해당 데이터는 실행파일에 기록이 되는데 초기값을 가지지 않는 bss의 경우 초기값을 저장할 필요가 없어 용량을 줄일 수 있음.
#### 데이터 타입
- byte = 1
- word = 2
- dword = 4
- qword = 8
#### 보수법
- 가장 최상위 비트는 부호 비트로 사용한다.
	1. byte = 0b1010 0000 = -128 + 32
- 모든 비트를 반전 + 1
	ex) 0b1011 = 0b0101
#### Enum vs Const
```cpp
const int ATK = 0
const int DEF = 1
const int CRT = 2

enum.TYPE {
	ATK,
	DEF,
	CRT
};

// const의 경우 경우에 따라 혹은 컴파일러에 따라 메모리를 할당할 수 있지만 enum의 경우 메모리를 할당하지 않고 컴파일러가 컴파일 단계에서 값을 교체하여 컴파일해준다.
// 따라서 const 보다는 enum을 사용하는 것이 메모리적으로 이득이다.

// 혹은 전처리문
#define DEF_ATK 0
#define DEF_DEF 1
#define DEF_CRT 2
```
#### 함수 호출
- 스택 프레임 
	1. 함수가 호출될 때 사용되는 스택 메모리(esp : stack pointer)
	2. 매게 변수 + 반환 주솟값 + 지역 변수
#### 포인터
```cpp
void test(const Test* t) {
	// t에 저장되어있는 주솟값은 수정 가능하다.
	// t에 저장된 주솟값의 값은 수정 불가능하다.
}

void test(Test* const t) {
	// t에 저장되어있는 주솟값을 변경할 수 없다
	// t에 저장된 주솟값의 값은 수정 가능하다.
}
```
#### Class
- 생성자
```cpp
class Test {
// 초기화 리스트 사용 생성자
// 초기화 리스트에서는 객체가 포함하고 있는 객체를 생성할때 기본 생성자를 호출한다
// 본인이 다른 생성자를 호출하고 싶다면 명시적으로 초기화 리스트에 써주자!
// 그 외에도 const 값 초기화, 참조 값 초기화에서도 사용한다
Test(float x) : f(x) {}

// 복사 생성자
Test(const Test& other) {}

// 타입 변환 생성자
// 인자를 하나만 받는 경우
// 암시적(explicit) 형변환이 일어남
Test(int a) {}

// 타입 변환 생성자
// 인자를 하나만 받는 경우
// 암시적인(explicit) 형변환을 막는다.
explicit Test(int a) {}
};
```
1. 상속 시 생성자 호출은 부모 -> 자식 순으로 호출된다. (어셈을 보면 자식 생성자가 부모 생성자를 호출한다.)
```cpp
class A {
	A() {}
	A(int a) {}
};

class B {
	// 부모의 특정 생성자를 호출하고 싶을때 아래와 같이 사용
	B(int a) : A(a) {}
};
```
- 소멸자
```cpp
class Test {
	// 소멸자를 가상함수로 구현하는 이유는 부모 타입으로 타입 캐스팅이 일어났을때 해당 객체를 delete하면 부모 형태의 소멸자만 호출함.
	// 따라서 자식의 소멸자를 호출하기 위해서 가상 함수로 선언한다.
	virtual ~Test() {}
}
```
- 얕은 복사(Shallow copy) and 깊은 복사(Deep copy)
	
- 특징
	상속성
	은닉성(캡슐화)
		건드리면 위험한 데이터를 관리할때
		다른 방식으로 제어하길 원할때
	다형성
		오버로딩 : 함수 중복 정의
		오버라이드 : 함수 재정의
		가상함수
			가상 테이블
				가상 함수를 선언하는 순간 가상 테이블의 주솟값을 가르키는 배열 변수를 하나 선언한다. (클래스의 크기 증가)
				동적 바이딩
				정적 바인딩
			순수 가상 함수
- Operator overloading
```cpp
class Test {
// 전위 증감
// 전위 증감의 경우 본인을 반환하기 때문에 체인 호출이 가능함
Test& operator++() {}

// 후위 증감
// 후위 증감은 본인의 복사값만 전달해주게 만든다.
Test operator++(int) {}
};

// 대입 연산자 오버로딩
Test& operator=(const Test& other) {}
```
#### 동적 할당
- malloc/free는 함수, new/delete는 연산자
- 차이
	new/delete의 경우 생성 타입이 클래스인 경우 생성자와 소멸자를 호출해준다.
- double free
	여러번 delete 혹은 free를 호출한 경우 -> 에러
#### 타입 변환
- 종류
	값 변환 : 비트열 재구성 (ex float <-> int)
		`int a = (float)b`
	참조 타입 변환 : 비트열을 재구성하지 않음
		`int a = (float&)b`
- 안전도
	안전한 변환 = 작은 크기의 타입 -> 큰 크기의 타입 (ex int -> long long)
	불안전한 번환 = 큰 크기의 타입 -> 작은 크기의 타입 (ex long long -> int) 혹은 완전히 다른 타입
- 의도
	암시적 변환 : 컴파일러가 자동으로 변환
	명시적 변환 : 프로그래머가 직접 타입 변환
	```cpp
	class Test {
	};
	class Test1 {
		// 타입 변환 생성자
		Test1(const Test& test) {}
		
		// 타입 변환 연산자
		operator Test() {} 
	};
	```
- 상속 관계
	자식 -> 부모 = 가능
	부모 -> 자식 = 불가능
- 메모리 참조의 위험이 있으면 그냥은 해주지 않는다!
- 캐스팅 연산자
	static_cast
		상식적인 타입 캐스팅만 허용 (ex. int <-> float)
		다운 캐스팅, 업 캐스팅
		안전성 보장 x
	dynamic_cast
		상속 관계 상의 안전한 타입 캐스팅(virtual 함수를 선언해야 한다)
		잘못된 타입 반환 시 nullptr 반환
	const_cast
		const를 붙이거나 떼어내는 캐스팅
	reinterpret_cast
		포인터와 관계 없는 전혀 다른 타입 변환
		가장 위험하고 강력한 캐스팅
#### 가변 인자
```cpp
template <typename T, typename ...TArgs>
void AddComponent(const Entity& entity, TArgs&& ...args) {
	T newComponent(std::forward<TArgs>(args)...);
}
```
#### Smart pointer
###### 종류
```cpp
shared_ptr<T> sp = std::make_shared<T>();
unique_ptr<T> up = std::make_unique<T>();
weak_ptr<T> wp;

// 스마트 포인터를 형변환 할때는 std::static_pointer_cast<?>()를 사용할 것
// unique_ptr<>의 주인 변환은 std::move()로 가능
```
###### 조심해야 할 점
shared_ptr의 서로 참조
```cpp
int main() {
	shared_ptr<Test> a;
	shared_ptr<Test> b;
	a->_target = b;
	b->_target = a;
	// 영원히 소멸되지 않음
}
```
###### 컴파일 에러
```cpp
class T {
	unique_ptr<class T> ptr; // 전방 선언을 할 경우 컴파일에러
}
```
#### 함수 포인터
###### 코드
```cpp
typedef int(*FUNC_NAME)(FUNC_ARGS...);

void test() {}

int main() {
	FUNC_NAME func = &test;
}
```
###### 확장
함수 포인터의 문제점 해결
전역 함수 및 정적 함수만 담을 수 있다. (class의 멤버 함수는 `this*`를 담아야 하니깐)
- 해결
	멤버함수 포인터
	```cpp
	class Test {
	public:
		void hello() {}
	};
	
	typedef int(Test::*FUNC_NAME)(FUNC_ARGS...);

	int main() {
		Test test;
		FUNC_NAME func = &Test::hello;
		(test.*func)(); // 일반 호출
	}
	```

함수 객체
함수 포인터는 형식이 맞지 않으면 사용할 수 없다
상태를 가질 수 없다
- 해결
	``` cpp
	class Functor {
	public:
		RET_TYPE operator()(FUNC_ARGS...) {}
		// 여러 버전 오버로딩 가능
	};

	int main() {
		Functor functor;
		functor(); // 일반 호출
	}
 	```
#### Template
```cpp
template<typename T>
void test(T a) {
	// ...
}

// specification template
template<>
void test(float a) {
	// ...
}

template<typename T, int SIZE>
class Test {
	int arr[SIZE];
};

// specification template
template<int SIZE>
class Test<float, SIZE> {

};

void main() {
	// 아래 코드 실행시 Test<10>과 Text<20>의 완전히 다른 클래스 타입으로 인식
	Test<10> t1;
	Test<20> t2;
}
```
#### Callback
특정 상황이 일어나면 해당하는 함수 호출을 알려주는 개념
```cpp
class Functor1 {
public:
	void operator()() {}
};

class Functor2 {
public:
	void operator()() {}
};

template<typename T>
void test(T fuctor) {
	// ...
}

void main() {
	Functor1 f1;
	Functor2 f2;

	test(f1);
	test(f2);
}
```
#### STL
###### vector
정의 : 동적 배열
존재 이유 : 고정인 배열은 불편한 경우가 있다
특징
- 초기에 여유분을 두고 할당한다
- 여유분까지 찬 경우 메모리를 늘린다
궁금증
- 초기에는 얼마나 여유분을 두는가?
	특정하게 지정해주지 않는한 꽉 차게 초기 사이즈를 둔다\
- 꽉 찬 경우 얼마나 늘려야 할까?
	일반적으로 약 1.5배 씩 증가하는 걸 확인함
- 기존의 데이터는 어떻게 처리할까?
삽입, 삭제
- o(n)
처음/끝 삽입 삭제
- 처음 삽입/삭제 = o(n)
- 끝 삽입 삭제 = o(1)
임의 접근
- o(1)
삭제 코드
```cpp
for(auto itr = arr.begin(); itr != arr.end(); itr++) {
	if(*itr == 3) {
		itr = arr.erase(itr);
		--itr;
	}
}
```
###### list
#### rvalue Reference
왼쪽 값(lvalue) : 값을 할당할 수 있는 지속되는 객체
오른쪽 값(rvalue) : lvalue가 아닌 나머지 값
```cpp
class Knight {
public:
	//...
	// 이동 생성자
	Knight(Knight&& other) { }
	// 대입 연산자
	Knight& operator=(const Knight& other) {
		// other의 값을 변경해서는 안됨
		// other에 대해 깊은 복사가 일어나야 함.
	}
	// 이동 대입 연산자
	Knight& operator=(Knight&& other) noexcept {
		// 어차피 이 함수가 끝나면 사라지는 객체임을 보장.
		// other의 값을 수정해도 상관 없음.
		// 얕은 복사가 일어나도 상관없음.
	}
};

void Test(Knight& knight) {} // 왼쪽값 참조 함수
void Test1(const Knight& knight) {}
void Test2(Knight&& knight) {} // 오른쪽값 참조 함수

int main() {
	Knight knight;

	Test(Knight()); // compile error. 임시 객체는 지속되지 않는 객체이기에 lvalue가 아님.
	Test1(Knight()); // ok. 값을 수정하지 않는 함수이기에 가능함.
	Test2(knight); // compile error. 오른쪽 참조값만 받는 함수이기에 불가능함.
	Test2(Knight()); // ok. 오른값 참조를 받는 함수이기에 가능함.
	Test2(static_cast<Knight&&>(knight)); // ok.

	Knight other;
	other = knight; // 대입 연산자 호출
	other = static_cast<Knight&&>(knight); // 이동 대입 연산자 호출
	other = std::move(knight); // 이동 대입 연산자 호출
}
```
###### Forwarding Reference(전달 참조)
c++17부터 사용
type guessing의 상황에서만 일어남
```cpp
template<typename T>
void Test(T&& parm) {
	// 오른쪽값으로 받았을 경우
	// 왼쪽값으로 받았을 경우
	Move(std::move(parm)); // not ok. 이 경우는 왼쪽값으로 받았을 경우 문제가 있다.
	Move(std::forward<T>(parm)); // ok. 왼쪽값, 오른쪽값에 따라 알아서 바꿔줌
}

int main() {
	Knight knight;

	Test(knight); // ok. 왼쪽 참조 함수로서 호출
	Test(std::move(knight)); // ok. 오른쪽 참조 함수로서 호출

	auto&& other = knight; // ok. 왼쪽 참조객체로 생성
	auto&& other = std::move(knight); // ok. 오른쪽 참조객체로 생성, cf) 이 경우 other는 왼쪽값이 된다
}
```
