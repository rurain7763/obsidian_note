- `docker pull image (image_name):(version_number/defualt = lastest)`
- `docker rmi (image_name)`
- `docker run --name (container_name) -d -p (port_number) -e (environment_value/ex)ROOT_PASSWORD=1234) (image_name)`
	1. -d : 백그라운드 실행. (데몬 모드)
	2. -e : 컨테이너의 환경 변수.
	3. -p : 포트 포워딩. container_port:host_port/protocol
	4. -t : 가상 터미널 환경 생성.
	5. --name : 컨테이너 이름.
	6. --rm : 컨테이너가 종료되면 자동 삭제.
	7. -i : 입력 대기.
- `docker commit (container_name) (image_name)`
	1. 해당 컨테이너 이미지로 만들기.
- 
