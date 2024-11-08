### Overview
#### Tokenizer
```c
int num = 5;
int res = 0;
while(num > 0) {
  res += num;
  num--;
}
```
해당 코드를 토큰화하면 다음과 같다.
```
[
	(TOK_IDENTIFIER, "int"),
	(TOK_IDENTIFIER, "num"),
	(TOK_EQ, "="),
	(TOK_NUMBER, "5"),
	(TOK_SEMICOLON, ";"),
	(TOK_IDENTIFIER, "int"),
	(TOK_IDENTIFIER, "res"),
	(TOK_EQ, "="),
	(TOK_NUMBER, "0"),
	(TOK_SEMICOLON, ";"),
	(TOK_WHILE, "while"),
	(TOK_LPAREN, "("),
	(TOK_IDENTIFIER, "num"),
	(TOK_GT, ">"),
	(TOK_NUMBER, "0"),
	(TOK_RPAREN, ")"),
	(TOK_LBRACE, "{"),
	
	//...
]
```

**토큰 형식**
TokenType + Lexeme

**종류**
- `TOK_IDENTIFIER` : 변수, 함수, 클래스 등의 이름
- `TOK_NUMBER` : 숫자
- `TOK_EQ` : `=`
- `TOK_SEMICOLON` : `;`
- `TOK_WHILE` : `while`
- `TOK_LPAREN` : `(`
- etc
#### AST
> AST(Abstract Syntax Tree)는 컴파일러가 소스코드를 분석하여 만드는 트리 구조의 자료구조이다. 이를 통해 컴파일러는 소스코드를 분석하고 실행 가능한 코드로 변환한다.

```
Stmts ([
	Assignment(Ident("num"), Number(5)),
	Asssgnment(Ident("res"), Number(0)),
	WhileStmts(
		BinaryOp(Ident("num"), Number(0), GT),
		Stmts ([
	      Assignment(Ident("res"), BinaryOp(Ident("res"), Ident("num"), ADD)),
	      Assignment(Ident("num"), BinaryOp(Ident("num"), Number(1), SUB))
	    ])
	)
])
```

**종류**
- `Assignment` : 변수 할당
- `WhileStmts` : while문
- `BinaryOp` : 이항 연산
- `Ident` : 변수 이름
- `Number` : 숫자
- etc

**장점**
- TypeCheck 가능
- 최적화 가능
- 코드 생성 가능
#### Interpreter
> AST를 통해 해당 CPU의 instruction set에 맞게 코드를 생성하고 실행한다.

ex)
``` asm
.data
num: .word 5
res: .word 0

.text
main:
  lw $t0, num
  lw $t1, res
  loop:
    add $t1, $t1, $t0
    sub $t0, $t0, 1
    bgt $t0, 0, loop

//...
```
#### Implementation
##### Lexer
> 소스코드를 토큰화한다.
###### Scanning algorithm
```c
char str[] = "int num = 5;";
int start = 0;
int curr = 0;
for(int i = 0; i < strlen(str); i++) {
  char c = str[i];
  if(isDigit(c)) { 
	  if(!isDigit(str[i + 1])) {
		  // add token
	  }
  }
  else if(isOperator(c)) { //... }
  // ...
  else if(c == ' ') {
	  start++;
	  cur++;	  
  }
}
```
