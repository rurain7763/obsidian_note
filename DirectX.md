#### 정의
- gpu를 제어하기 위한 저수준 그래픽 API
####  렌더링 파이프 라인
- cpu : 직렬처리
- gpu : 병렬처리
- cpu가 gpu에게 결과물을 출력하라는 명령을 주기 위한 단계
- vertex shader -> resterizer -> pixel shader -> output merger
![pipeline](http://www.braynzarsoft.net/image/100202)
#### Graphics pipeline
###### Input assembler
그리기에 필요한 리소스(vertex buffer, index buffer, topology, layout etc) 로드
###### vertex shader
점에 대한 연산 프로그램 실행
###### tessellation : hull + domain
###### geometry shader
###### rasterizer
도형별 내부 픽셀 계산 
###### pixel shader
각 픽셀에 대한 연산 프로그램 실행
###### output merger
깊이 테스트, 블렌딩
#### 동작 과정
![process](http://www.braynzarsoft.net/image/100204)
#### DXGI
- 여러 그래픽 API 중에서도 공통적으로 사용되는 API를 가지는 라이브러리. (ex. direct2d에서나 direct3d에서 둘 다 swapchain은 필요하다.)
- `IDXGIFactory` : swapchain 생성과 adaptor 열거에 쓰임.
#### ComPtr
- component object model
- IUnknown 클래스 상속
- 객체의 Refrence count를 관리하는 스마트 포인터
- 메모리 해제를 위한 `Release()`호출
- `Get()` : 가리키는 포인터 반환.
- `GetAddressOf()` : 해당 ComPtr를 가르키는 포인터 반환.
#### Depth Buffer
- 픽셀의 z 값을 관리하는 깊이 버퍼
- 0~1사이의 값을 가진다.
- 후면 버퍼와의 위치와 크기가 1대 1 대응이다. (후면버퍼 1280x720이면 깊이 버퍼도 1280x720)
- 하나의 텍스쳐로 이루어짐.
![db](https://learn.microsoft.com/en-us/windows/uwp/graphics-concepts/images/zbuffer.png)
#### Device
- `D3D12CreateDevice()`
#### SwapChain
- 이중 버퍼링(삼중, 사중도 가능)를 관리하는 인터페이스.
- 전면 버퍼, 후면 버퍼를 번갈아가며 사용.
- 버퍼는 당연히 하나의 텍스쳐.
- `ResizeBuffers()`
- `Present()`

**DXGI_SWAP_CHAIN_DESC**
SwapEffect
- DXGI_SWAP_EFFECT_DISCARD :  비트 블록 전송 방식/버퍼를 버림
- DXGI_SWAP_EFFECT_FLIP_DISCARD : 비트 블록 전송 방식/버퍼를 버리고 새로 만듬
- DXGI_SWAP_EFFECT_FLIP_SEQUENTIAL :  대칭 이동 전송 방식/버퍼를 버리지 않음
- DXGI_SWAP_EFFECT_SEQUENTIAL : 대칭 이동 전송 방식/버퍼를 버리지 않음

**비트 블록 전송 방식**
백 버퍼를 복사하여 전면 버퍼로 전송하는 방식
버퍼 카운터가 하나만 있으면 됨.

**대칭 이동 전송 방식**
백 버퍼의 포인터를 전면 버퍼로 전송하는 방식
버퍼 카운터가 두 개 필요함.
v-sync를 사용할 때 사용.
#### Vertex Buffer
정점의 배열
#### Layout Input
정점 배열의 구조
#### Command List
- 하나의 command allocator를 가진다.
#### Command Queue
- `CreateCommandQueue()`
- Type
	1. Direct command queue : 기본 큐. 모든 명령어를 받아들임
	2. Compute command queue : compute 와 copy 명령어만 받아들임.
	3. Copy command queue : copy 명령어만 받아들임.
- 
#### Command Allocator
- command list에 저장되어 있는 명령들이 저장되어 있는 공간.
#### Resources
- Resource Type
	1. Texture
	2. Buffers
- Resource Ref/View
	1. Constant buffer view
	2. Unordered access view
	3. Shader resource view
	4. Samplers
	5. Render target view
	6. Depth stencil view
	7. Index buffer view
	8. Vertex buffer view
	9. Stream output view
#### Texture
- 1차 배열로 이루어진 1차원 텍스쳐, 2차 배열로 이루어진 2차원 텍스쳐, 3차 배열로 이뤄진 3차원 텍스쳐가 있음.
- 모든 텍스쳐가 색깔 데이터 만을 담는 것은 아님. (ex. 법선 맵핑)
- 데이터 종류 [msdn](https://learn.microsoft.com/en-us/windows/win32/api/dxgiformat/ne-dxgiformat-dxgi_format)
	1. DXGI_FORMAT_R32G32B32_FLOAT : 각 원소는 32비트 부동 소숫점 성분 3개로 이루어짐.
	2. DXGI_FORMAT_R16G16B16A16_UNORM : 각 원소는 0~1사이로 패킹되는 16비트 성분 4개로 이루어짐.
	3. DXGI_FORMAT_R32G32_UINT : 각 원소는 부호 없는 32비트 정수 2개로 이루어짐.
- 
#### Descriptor
- 셰이더가 자원을 어디에 있고 어떻게 해석해야 하는지 기술한 객체.
- 렌더링 파이프라인에 바인딩할 때 자원이 직접 바인딩되는 것이 아님.
- View라는 단어와 동의어로 사용한다.
- 종류
	1. CBV : 상수 버퍼 (constant buffer)
	2. SRV : 셰이더 (shader resource)
	3. UAV : 순서 없는 접근(?) (unordered access view)
	4. sampler : 텍스쳐 자원
	5. RTV : 렌더링 대상 (render target)
	6. DSV : 깊이, 스텐실 자원 (depth/stencil)
#### Descriptor Table
#### Descriptor Heap
- Descriptor table을 저장하는 공간.
- 서술자 종류마다 개별적으로 필요.
#### Root Signature
#### Work Flow
![workflow](http://www.braynzarsoft.net/image/100211)