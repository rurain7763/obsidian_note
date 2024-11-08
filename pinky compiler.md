### Overview
###### Pinky language
[사이트](https://pinky-lang.org/)
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
> 소스코드를 토큰화한다. 문법은 신경쓰지 않는다. (있으면 안되는 문자나 잘못된 입력은 예외)
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
###### 코드
```python
class Lexer:
    def __init__(self, source):
        self.source = source
        self.start = 0 # 토큰 시작 인덱스
        self.curr = 0 # 현재 인덱스
        self.line = 1 
        self.tokens = []
        pass

	# 현재 curr 위치의 문자 반환 후 curr 증가
    def advance(self):
        ch = self.source[self.curr]
        self.curr = self.curr + 1
        return ch

	# 현재 curr 위치의 문자 반환
    def peek(self):
        return self.source[self.curr]

	# curr 위치의 다음 문자 반환
    def lookahead(self, n = 1):
        return self.source[self.curr + n]

	# 현재 문자와 expected가 같은지 확인 후 같으면 curr 증가 후 True 반환
    def match(self, expected):
        if self.source[self.curr] != expected:
            return False
        self.curr = self.curr + 1
        return True

	# 현재 start(포함) - curr(미포함) 까지의 문자를 tokens에 추가
    def add_token(self, token_type):
        self.tokens.append(Token(token_type, self.source[self.start:self.curr], self.line))

	# source를 읽어 토큰화한 후 tokens 반환
    def tokenize(self):
        while self.curr < len(self.source):
            self.start = self.curr
            ch = self.advance()
            if ch == '+': self.add_token(TOK_PLUS)
            elif ch == '-': self.add_token(TOK_MINUS)
            elif ch == '*': self.add_token(TOK_STAR)
            elif ch == '/': self.add_token(TOK_SLASH)
            # etc
        
        return self.tokens
```
###### 개선점
현재 토큰화 클래스는 토큰에 새로운 string을 할당하고 있는데 이는 메모리 누수를 일의킬 수 있다. 따라서 c/c++ 같이 포인터를 활용할 수 있는 언어로 제작 시 해당 토큰이 가르키는 원본 문자열의 시작 주소와 길이를 저장하는 방식으로 개선할 수 있다.
#### Parser
> 문법 체크 및 AST 생성
###### CFG(Context-Free Grammar)
###### BNF() 표기법
