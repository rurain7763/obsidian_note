#### Shell
- DB 접속
	`mysql -h(server_ip) -u(user_id) -p`
	 이후 password 입력.
- Excute sql with file
	 `source (path_to_sql_file)`
	 ./ : 현재 terminal 위치.
- 
#### Event 
- 주기적으로 DB에 해줘야하는 작업.
- 이벤트 스케쥴러가 켜져있는지 확인.
	`show variables like "event%";`
	 꺼져있는 경우 : `set global event_scheduler = "ON";`
- 생성되어 있는 이벤트 확인.
	`select * from information_schema.events;`
- 이벤트 삭제.
	`drop event (event_name);`
- 이벤트 생성.
	```
	create event
		(event_name)
	on schedule every 1 day
	starts
		"xxxx-xx-xx xx:xx:xx"
	comment
		(comment_text)
	do
		(query)
	```
- 