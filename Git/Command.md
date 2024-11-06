- 브랜치 리스트 새로고침
	: `git fetch --prune`
#### Error 처리
- `# Git Error: RPC failed; HTTP 400 curl 22 The requested URL returned error: 400`
	한번에 많은 파일을 push하면 뜨는 에러
	해결 방법 : `git config --global http.postBuffer 524288000` 입력 이 후 다시 push
- 
