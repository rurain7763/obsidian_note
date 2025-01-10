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
현재 model, view, projection matrix를 현산을 매 프레임당 계산하고 있음. 이를 개선해야함.
충돌 체크 시 모든 entity에 대해 체크하고 있음. 공간 분할을 통해 개선해야함.
텍스쳐의 속성을 개별적으로 변경하지 못함. 개선해야함.