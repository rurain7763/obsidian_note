#### Editor mode
`i` : 커서 앞쪽으로 입력모드
`a` : 커서 뒤쪽으로 입력모드
`I` : 줄의 맨 앞쪽으로 입력모드
`A` : 줄의 맨 뒤쪽으로 입력모드
`o` : 아래에 새로운 줄 시작 후 입력모드
`O` : 위에 새로운 줄 시작 후 입력모드
`h, j, k, l` : 커서 이동
`u` : undo
`ctrl + r` : redo
`v` : visual mode로 들어가기
`V` : line 선택 visual mode로 들어가기
`p` : 붙여넣기
`dd` : 라인 잘라내기
`D` : 해당 커서 뒤의 모든 글자 삭제
`yy` : 줄 복사
`w` : 다음 단어로 커서 이동
`b` : 이전 단어로 커서 이동
`d + w` : 단어 삭제 (커서를 기준으로 단어 삭제)
`d + i + w` : 단어 삭제 (커서가 어디있든 관계없이 단어 삭제)
`d + 0` : 현재 커서에서 라인의 처음까지 지우기
`d + $` 현재 커서에서 라인의 끝까지 지우기
`y + i + ?` : ? 사이의 모든 글자 복사
`gg` : 파일 처음으로 이동
`G` : 파일 끝으로 이동
`? + g` : 특정 라인으로 이동
`ctrl + v` : 멀티 커서
`ma'` : 웨이 포인트 설정
`'a` : 웨이 포인트 이동
`zz` : 현재 커서가 가르키는 라인을 페이지 중앙 배치
`.` : 마지막 명령어 반복
`" + 레지스터 번호 + p` : 해당 레지스터리에 저장된 글자를 붙여넣는다
`"+y` : 선택 라인 시스템 클립보드로 복사
#### Visual mode
`d` : 삭제
`y`  : 복사
#### Terminal mode
`:set number` : 라인 번호 보기
`:set mouse=a` : 마우스 활성화
`/단어` : 특정 단어 찾기
`n` : 다음 단어
`u` : 이전 단어
`%s/타겟 글자/바꿀 글자/g` : 글자 교체
`:reg` : 클립 보드
`:vsplit (파일 경로)` : 멀티 윈도우로 보기
#### .vimrc
vim config 파일

``
