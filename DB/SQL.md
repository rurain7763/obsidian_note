- schema 선택
	`use (name_of_schema);`
- 전체 테이블 보기
	`select * from (table_name);`
- DB 생성
	`create (database_name);`
- DB 리스트
	`show databases;`
- DB 제거
	`drop (database_name);`
- Table 보기
	`show tables;`
- 현재 유저 Grant 보기
	 `show grants for CURRENT_USER();`
- 테이블 갯수 조회
	 `select count(*) from information_schema.tables where table_schema = '(schema_name)'`
- LEFT JOIN
	1. A 와 B를 합칠 때 A의 속성은 모두 포함하고 B의 속성은 A의 `key`값과 일치하는 것만 표시.
	2. `select (column) from A left join B on A.(column) = B.(column)`
- 테이블 속성 보기
	1. `describe (table_name)`
- 비번 바꾸기
	1. [Link](https://stackoverflow.com/questions/36099028/error-1064-42000-you-have-an-error-in-your-sql-syntax-want-to-configure-a-pa)
- 현재 시간 확인
	1. `select now();`
- Key 컬럼 관리
	1. 외래키로 참조하고 있는 값의 무결성을 유지할려면 CASCADE를 찾아 볼 것.
- 