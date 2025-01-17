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
#### TODO
현재 model, view, projection matrix를 현산을 매 프레임당 계산하고 있음. 이를 개선해야함. (precalculation system 구현)
충돌 체크 시 모든 entity에 대해 체크하고 있음. 공간 분할을 통해 개선해야함.
텍스쳐의 속성을 개별적으로 변경하지 못함. 개선해야함. (opengl과 directx의 차이 때문에)
collider가 움직이지 않는 버그
엔진 종료 시 더블 프리가 발생하는 버그
- 현재 엔티티를 매번 루프하여 시스템을 돌리고 있다. 만약에 전 프레임과 현재 프레임의 엔티티에 차이가 없다면 기존에 존재하는 뷰를 사용해서 루프를 돌게 하면 성능 향상이 될것 같다.
- 인스터스 드로잉으로 드로우 콜을 줄여야함.
- 에셋에 대한 정의가 불명확함.
- 매번 트랜
