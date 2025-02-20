#### Logger
#### Assertion
#### Platform Layer
###### Windowing
#### Application Layer
#### Memory Subsystem
#### Event System
###### TODO
멀티 스레드를 활용하여 작업 큐 형태의 이벤트 시스템 구현해야함
Event의 우선 순위를 정할 수 있어야함
#### Input System
#### Renderer System
###### vulkan
**instance 생성**
**device 생성**
- physical device 선택
- logical device 생성
**swapchain 생성**
render pass 생성
command buffer 생성
#### Material
###### dynamic material(동적 재질)
###### GPGPU
General Purpose Graphic Processing Unit
단순 작업이 여러개 일때 유용
gpu에서 데이터를 처리하므로 데이터 전달 작업이 필요없다.
###### CUDA
NVIDIA에서 개발한 GPGPU를 위한 툴킷
###### Compute Shader
GPU에서 병렬로 처리되는 프로그램

**directx11**
u(unordered register)로 시작하는 레지스터리에 바인드
unordered access view에 바인드를 하면 쓰기가 가능하다.
#### Particle system
###### billboard
#### Instancing drawing
#### Tesellation Shader
#### Geometry Shader
#### Compute Shader
###### atomic operation
#### Noise
###### Paper burning effect
#### Post processing
#### Prefab asset
#### Clone game object
#### ImGUI
###### docking이 화면을 벗어나면 생기는 문제 해결
#### AssetStore
###### Lazy loading
#### 계층구조
###### 주의점
`SetParent()` : 자식 중에 부모가 되려하면 막아야 한다.
`AddChild()` : 자신의 부모 중에 하나를 자식으로 하려고 하면 막아야 한다.
#### Reflection
#### Serialization
#### Hot reload
#### Level loading
#### TODO
- 현재 model, view, projection matrix를 현산을 매 프레임당 계산하고 있음. 이를 개선해야함. (precalculation system 구현) - done
- 충돌 체크 시 모든 entity에 대해 체크하고 있음. 공간 분할을 통해 개선해야함.
- 텍스쳐의 속성을 개별적으로 변경하지 못함. 개선해야함. (opengl과 directx의 차이 때문에)
- collider가 움직이지 않는 버그 - done
- 엔진 종료 시 더블 프리가 발생하는 버그 - done
- 현재 엔티티를 매번 루프하여 시스템을 돌리고 있다. 만약에 전 프레임과 현재 프레임의 엔티티에 차이가 없다면 기존에 존재하는 뷰를 사용해서 루프를 돌게 하면 성능 향상이 될것 같다. - done
- 인스터스 드로잉으로 드로우 콜을 줄여야함.
- 에셋에 대한 정의가 불명확함. - done
- 매번 월드 매트릭스 및 뷰, 프로젝션 매트릭스를 구하는데 전 프레임과 현재 프레임의 엔티티에 변화가 없다면 기존의 값을 사용하게 하면 성능 향상이 될것 같다.
- 셰이더에 매트릭스를 넘겨줄때 미리 곱한 매트릭스를 넘겨주면 성능 향상이 될것 같다.
- 렌더링 시스템에 대해 2d, 3d 구분이 필요함.
- 현재 opengl에 대한 렌더링이 지체되고 있음. 시간이 나면 directx와 같이 미구현 된 부분을 구현해야함.
- StructedBuffer를 Map, Unmap을 통해 구현하면 성능 향상이 될것 같다. - done
- rect transform 추가 구현 (anchor, pivot을 기반으로 위치 및 크기 조정)
- Pipeline 코드에 대한 모호성이 조금 있는것 같음. 코드를 다시 한번 확인해야함. - done
- image를 불러올때 rg 혹은 grayscale인지 사용자가 구분하게 해야함. (현재는 무조건 channel이 2개면 grayscale로 간주)
- 2d spot light 추가 구현
- 에셋 참조를 경로로 하는 것이 아니라 guid로 하는 것이 좋을것 같다.
- outliner ui를 게임 오브젝트가 갱신될때 refresh하는 로직으로 변경
#### Memo
- 에셋은 객체를 생성하는 틀 역할을 하는것도 좋을것 같음.
