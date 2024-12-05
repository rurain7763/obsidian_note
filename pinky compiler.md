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
> 문법을 표현하는 방법

**냉장고 재료로 요리하기:**

냉장고에 여러 재료가 있다고 생각해 봅시다. Context-free grammar는 이 재료들을 이용해서 요리하는 레시피와 같습니다.

- **재료:** 계란, 베이컨, 빵, 치즈 (단어처럼 문장의 기본 요소)
- **레시피 (규칙):**
    - 아침 식사 --> 계란 요리 + 빵
    - 계란 요리 --> 스크램블 에그 | 프라이
    - 빵 --> 토스트 | 베이글

이 레시피를 따라하면 다양한 아침 식사를 만들 수 있습니다.

- 스크램블 에그 + 토스트
- 프라이 + 베이글
- 스크램블 에그 + 베이글
- 프라이 + 토스트

중요한 점은, "계란 요리"를 만들 때 "빵"이 무엇인지는 신경 쓰지 않아도 된다는 것입니다. 스크램블 에그를 만들든 프라이를 만들든, 빵이 토스트인지 베이글인지는 상관없습니다. 각 부분을 독립적으로 만들 수 있습니다. 이것이 바로 "context-free (문맥 자유)" 의 의미입니다. 주변 상황(문맥)에 상관없이 각 부분을 독립적으로 만들 수 있다는 뜻입니다.

Context-free grammar는 이처럼 문장을 구성하는 규칙을 정의하는 방식입니다. 각 규칙은 다른 규칙과 상관없이 독립적으로 적용될 수 있습니다.
###### BNF(Backus-Naur Form) 표기법
> CFG를 표기하는 방법 중 하나로, `::=`, `|`를 사용하여 정의한다.

**BNF 표기**
```
<expr> ::= <term> (('+' | '-') <term>)*
<term> ::= <factor> (('*' | '/') <factor>)*
<factor> ::= <unary>
<unary> ::= ('+' | '-' | '~') <unary> | <primary>
<primary> ::= <integer> | <float> | <grouping> | <bool> | <string>
<grouping> ::= '(' <expr> ')'
```
참고 영상 : [video](https://courses.pikuma.com/courses/take/create-a-programming-language-compiler/lessons/59251675-grammar-for-simple-expressions)
###### Helper method
```python
# 현재 토큰 반환
def peek(self):
	return self.tokens[self.curr]

# 현재 토큰 반환 후 curr 증가
def advance(self):
	token = self.peek()
	self.curr = self.curr + 1
	return token

# 현재 토큰의 token_type이 expected와 같은지 확인
def check(self, expected):
	if self.curr >= len(self.tokens): return False
	return self.peek().token_type == expected

# 현재 토큰의 token_type이 expected와 같은지 확인 후 같으면 curr 증가, 그렇지 안다면 예외 발생
def expect(self, expected):
	if self.curr >= len(self.tokens):
		raise SyntaxError(f'Expected {expected!r}, found EOF')
	elif self.peek().token_type != expected:
		raise SyntaxError(f'Expected {expected!r}, found {self.peek().token_type!r}')
	else:
		return self.advance()

# 현재 토큰의 token_type이 expected와 같은지 확인 후 같으면 curr 증가, 그렇지 않다면 False 반환
def match(self, expected):
	if self.curr >= len(self.tokens) or self.peek().token_type != expected: return False
	self.curr = self.curr + 1
	return True

# 이전 토큰 반환
def previous_token(self):
	return self.tokens[self.curr - 1]
```
###### Expression
```python
class Expr(Node):
    # x + (3 * y)
    pass

class Integer(Expr):
    # 123
    def __init__(self, value):
        assert isinstance(value, int), value
        self.value = value

class Float(Expr):
    # 3.141592
    def __init__(self, value):
        assert isinstance(value, float), value
        self.value = value

# string, bool, etc...

class UnOp(Expr):
    # -x
    def __init__(self, op : Token, operand : Expr):
        assert isinstance(op, Token), op
        assert isinstance(operand, Expr), operand
        self.op = op
        self.operand = operand

class BinOp(Expr):
    # x + y
    def __init__(self, op : Token, left : Expr, right : Expr):
        assert isinstance(op, Token), op
        assert isinstance(left, Expr), left
        assert isinstance(right, Expr), right
        self.op = op
        self.left = left
        self.right = right

# LogicalOp, etc...

class Grouping(Expr):
    # ( <Expr> )
    def __init__(self, value):
        assert isinstance(value, Expr), value
        self.value = value
```

**코드**
```python
def primary(self):
	if self.match(TOK_INTEGER): return Integer(int(self.previous_token().lexeme))
	elif self.match(TOK_FLOAT): return Float(float(self.previous_token().lexeme))
	elif self.match(TOK_LPAREN):
		expr = self.expr()
		if self.match(TOK_RPAREN): return Grouping(expr)
		else: raise SyntaxError(f'Error: ")" expected.')

def unary(self):
	if self.match(TOK_PLUS) or self.match(TOK_MINUS) or self.match(TOK_NOT):
		op = self.previous_token()
		operand = self.unary()
		return UnOp(op, operand)
	else:
		return self.primary()

def factor(self):
	return self.unary()
	
def term(self):
	expr = self.factor()
	while self.match(TOK_STAR) or self.match(TOK_SLASH) or self.match(TOK_CARET):
		op = self.previous_token()
		right = self.factor()
		expr = BinOp(op, expr, right)
	return expr

def expr(self):
	expr = self.term()
	while self.match(TOK_PLUS) or self.match(TOK_MINUS):
		op = self.previous_token()
		right = self.term()
		expr = BinOp(op, expr, right)
	return expr
```
**위 코드는 +,-, x, /만 적용된 코드임**
###### Statement
> 프로그램은 여러 statement로 구성되어 있다. (if, while, assign, print, etc)

```python
class Stmt(Node):
    # perform action (while, if, assign, etc)
    pass

class PrintStmt(Stmt):
    def __init__(self, value : Expr, line):
        assert isinstance(value, Expr), value
        self.value = value
        self.line = line

class WhileStmt(Stmt):
    pass

class Assignment(Stmt):
    pass

# IfStmt, etc...
```

```
print_stmt := 'print' expr

if_stmt := 
	'if' expr 'then' stmts
		('elif' expr 'then' stmts)*
		('else' stmts)?
	'end'
```

**코드**
```python
def print_stmt(self):
	if self.match(TOK_PRINT):
		val = self.expr()
		return PrintStmt(val, self.previous_token().line)

def stmt(self):
	if self.peek().token_type == TOK_PRINT:
		return self.print_stmt()
	#elif self.peek().token_type == TOK_IF:
	#    return self.if_stmt()
	#elif self.peek().token_type == TOK_FOR:
	#    return self.for_stmt()
	#elif self.peek().token_type == TOK_WHILE:
	#    return self.while_stmt()
	#elif self.peek().token_type == TOK_FUNC:
	#    return self.func_stmt()
	#else:
		# wtf

def stmts(self):
	stmts = []
	while self.curr < len(self.tokens):
		stmts.append(self.stmt())
	return Stmts(stmts, self.previous_token().line)
```
#### Interpreter
> AST를 통해 해당 CPU의 instruction set에 맞게 코드를 생성하고 실행한다. (모든 cpu instruction set을 다루기는 힘들기 때문에 이번 프로젝트에서는 python을 interface로 활용한다)
###### Environment
> 현재 블록에서 사용할 수 있는 변수들을 저장하는 공간 (함수 호출이나 if 블록 생성 시 새로운 environment 생성)
> Environment는 부모를 가지며, 부모의 변수에 접근할 수 있다.
```python
class Environment:
    def __init__(self, parent=None):
        self.vars = {}
        self.funcs = {}
        self.parent = parent
```
###### Expression
AST를 후위 순회하여 계산한다.
```python
def interpret(self, ast, env):
	if isinstance(ast, Integer):
		return float(ast.value)
	elif isinstance(ast, Float):
		return float(ast.value)
	elif isinstance(ast, Grouping):
		return self.interpret(ast.value, env)
	elif isinstance(ast, BinOp):
		left = self.interpret(ast.left, env)
		right = self.interpret(ast.right, env)
		if ast.op.token_type == TOK_PLUS: return left + right
		elif ast.op.token_type == TOK_MINUS: return left - right
		elif ast.op.token_type == TOK_STAR: return left * right
		elif ast.op.token_type == TOK_SLASH: return left / right
	elif isinstance(ast, UnOp):
		val = self.interpret(ast.operand)
		if ast.op.token_type == TOK_PLUS: return +val
		elif ast.op.token_type == TOK_MINUS: return -val
		elif ast.op.token_type == TOK_NOT: return not val
	elif isinstance(ast, LogicalOp):
		# and의 경우 왼쪽이 이미 false라면 오른쪽을 계산할 필요가 없고, or의 경우 왼쪽이 이미 true라면 오른쪽을 계산할 필요가 없다.
		left_type, left_value = self.interpret(ast.left, env)
		if (ast.op.token_type == TOK_AND and left_value) or (ast.op.token_type == TOK_OR and not left_value):
			return self.interpret(ast.right, env)
		else:
			return (left_type, left_value)
	# etc ...
```
###### Statement
```python
def interpret(self, ast, env):
	if isinstance(node, Stmts):
		for stmt in node.stmts:
			self.interpret(stmt, env)
	elif isinstance(node, PrintStmt):
		type, value = self.interpret(node.value, env)
		print(value)
	# etc ...
```
#### Variable
###### Identifier
```python
class Identifier(Expr):
    # x, PI, _a, start_val, etc...
    def __init__(self, name, line):
        assert isinstance(name, str), name
        self.name = name
        self.line = line
```
###### Assignment
```python
class Assignment(Stmt):
	# x := 4, y := x + 3, etc...
    def __init__(self, left : Expr, right : Expr, line):
        assert isinstance(left, Expr), left
        assert isinstance(right, Expr), right
        self.left = left
        self.right = right
        self.line = line
```
###### Parser
```python
else:
	left = self.expr()
    if self.match(TOK_ASSIGN):
        right = self.expr()
        return Assignment(left, right, self.previous_token().line)
```
###### Interpreter
interpret을 하여 현재 코드를 실행 경우 해당 environment에서 변수를 찾도록 한다.
```python
def interpret(self, ast, env):
	if isinstance(node, Assignment):
		r_type, r_value = self.interpret(node.right, env)
		env.set_vale(node.left.name, (r_type, r_value))
	elif isinstance(node, Identifier):
		val = env.get_value(node.name)
		if val == None:
		  runtime_error(f'Variable {node.name} is not defined.', node.line)
		elif val[1] == None:
		  runtime_error(f'Variable {node.name} is not initialized.', node.line)
		else:
		  return val
    elif isinstance(node, IfStmt):
		type, value = self.interpret(node.condition, env)
		if type != TYPE_BOOL:
			runtime_error('Condition is not a boolean expression', node.line)
			
		# 새로운 블록이 생성되었으므로 현재 환경을 부모로 가지는 새로운 환경을 생성하고 넘겨준다.
		if value:
			return self.interpret(node.then_stmts, env.new_env())
		else:
			return self.interpret(node.else_stmts, env.new_env())
    # etc ...
```
#### Loop
###### WhileStmt
```python
class WhileStmt(Stmt):
	# while <expr> do <stmts> end
    def __init__(self, condition : Expr, do_stmts : Stmts, line):
        assert isinstance(condition, Expr), condition
        assert isinstance(do_stmts, Stmts), do_stmts
        self.condition = condition
        self.do_stmts = do_stmts
        self.line = line
```
###### ForStmt
```python
class ForStmt(Stmt):
	# for <assignment>, <expr>, <expr>? do <stmts> end
    def __init__(self, assignment : Assignment, condition_val : Expr, step_val : Expr, do_stmts : Stmts, line):
        assert isinstance(assignment, Assignment), assignment
        assert isinstance(condition_val, Expr), condition
        assert step_val is None or isinstance(step_val, Expr), step_val
        assert isinstance(do_stmts, Stmts), do_stmts
        self.assignment = assignment
        self.condition_val = condition_val
        self.step_val = step_val
        self.do_stmts = do_stmts     
        self.line = line
```
###### Parser
```python
def while_stmt(self):
	self.expect(TOK_WHILE)
	condition = self.expr()
	self.expect(TOK_DO)
	do_stmts = self.stmts()
	self.expect(TOK_END)
	return WhileStmt(condition, do_stmts, self.previous_token().line)

def for_stmt(self):
	self.expect(TOK_FOR)
	assignment = self.assignment()
	self.expect(TOK_TO)
	condition_val = self.expr()
	if self.match(TOK_STEP): step_val = self.expr()
	else: step_val = None
	self.expect(TOK_DO)
	do_stmts = self.stmts()
	self.expect(TOK_END)
	return ForStmt(assignment, condition_val, step_val, do_stmts, self.previous_token().line)
```
###### Interpreter
```python
elif isinstance(node, WhileStmt):
	type, value = self.interpret(node.condition, env)
	if type != TYPE_BOOL:
		runtime_error('Condition is not a boolean expression', node.line)

	while value:
		self.interpret(node.do_stmts, env.new_env())
		type, value = self.interpret(node.condition, env)
elif isinsinstance(node, ForStmt):
	new_env = env.new_env()
	self.interpret(node.assignment, new_env)
	cond_type, cond_val = self.interpret(node.condition_val, new_env)
	if cond_type != TYPE_NUMBER:
		runtime_error('Condition is not a number expression', node.line)

	if node.step_val != None:
		step_type, step_val = self.interpret(node.step_val, new_env)
		if step_type != TYPE_NUMBER:
			runtime_error('Step is not a number expression', node.line)
	else:
		step_type = TYPE_NUMBER
		step_val = 1

	var_name = node.assignment.left.name
	cur_type, cur_val = new_env.get_value(var_name)
	if cur_val <= cond_val:
		# 현재 값이 조건 값보다 작다면
		step_val = self.expr()
		while cur_val <= cond_val:
			self.interpret(node.do_stmts, new_env.new_env())
			cur_type, cur_val = new_env.get_value(var_name)
			cur_val = cur_val + step_val
			new_env.set_vale(var_name, (cur_type, cur_val))
	else:
		# 현재 값이 조건 값보다 크다면
		if node.step_val == None: step_val = -1
		while cur_val >= cond_val:
			self.interpret(node.do_stmts, new_env.new_env())
			cur_type, cur_val = new_env.get_value(var_name)
			cur_val = cur_val + step_val
			new_env.set_vale(var_name, (cur_type, cur_val))
```
####  Function
###### model.py
```python
class Param(Decl):
	# <idetifier>
    def __init__(self, identifier, line):
        assert isinstance(identifier, Identifier), identifier
        self.identifier = identifier
        self.line = line

class FuncDecl(Decl):
	# func <identifier> "(" <params>? ")" <body_stmts> end
    def __init__(self, identifier, params, body_stmts, line):
        assert isinstance(identifier, Identifier), identifier
        assert all(isinstance(param, Param) for param in params), params
        self.identifier = identifier
        self.params = params
        self.body_stmts = body_stmts
        self.line = line

class FuncCall(Expr):
	# <identifier> "(" <expr>? ("," <expr>)* ")"
    def __init__(self, identifier, args, line):
        assert isinstance(identifier, Identifier), identifier
        assert all(isinstance(arg, Expr) for arg in args), args
        self.identifier = identifier
        self.args = args
        self.line = line

class FuncCallStmt(Stmt):
	# wrapper class for FuncCall
    def __init__(self, func_call : FuncCall):
        assert isinstance(func_call, FuncCall)
        self.func_call = func_call

class RetStmt(Stmt):
	# ret <expr>
    def __init__(self, value : Expr, line):
        assert isinstance(value, Expr)
        self.value = value
        self.line = line
```
###### Paser
```python
# 함수 정의
def func_decl(self):
	self.expect(TOK_FUNC)
	self.expect(TOK_IDENTIFIER)
	identifier = Identifier(self.previous_token().lexeme, self.previous_token().line)
	self.expect(TOK_LPAREN)

	max_params_cnt = 255
	curr_parmas_cnt = 0
	params = []
	while not self.match(TOK_RPAREN):
		self.expect(TOK_IDENTIFIER)

		# 파라미터가 max_params_cnt보다 많으면 에러 발생
		if curr_parmas_cnt >= max_params_cnt:
			parse_error(f"Functions can not have arguments more than {max_params_cnt}", self.previous_token().line)

		params.append(Param(Identifier(self.previous_token().lexeme, self.previous_token().line), self.previous_token().line))
		curr_parmas_cnt += 1
		self.match(TOK_COMMA)

	body_stmts = self.stmts()
	self.expect(TOK_END)
	return FuncDecl(identifier, params, body_stmts, self.previous_token().line)

# 함수 반환
def ret_stmt(self):
	self.expect(TOK_RET)
	return RetStmt(self.expr(), self.previous_token().line)
```
###### Interpreter
```python
elif isinstance(node, FuncDecl):
	env.set_func(node.identifier.name, (node, env))
elif isinstance(node, FuncCall):
	func, func_org_env = env.get_func(node.identifier.name)

	if func == None:
		runtime_error(f"Function {node.identifier.name} not declared", node.line)
	if len(func.params) != len(node.args):
		runtime_error(f"Function {node.identifier.name} expected {len(func.params)} arguments, but {len(node.args)} arguments were passed", node.line)

	new_env = func_org_env.new_env()
	for i in range(0, len(func.params)):
		new_env.set_value_as_local(func.params[i].identifier.name, self.interpret(node.args[i], env))

	# 함수 실행 부분
	# 함수 내부에서 ret을 만나면 Exception을 raise하여 이후의 모든 코드를 실행하지 않고 catch문을 만날때 까지 stack unwinding을 진행한다.
	try:
		self.interpret(func.body_stmts, new_env)
	except Return as e:
		return e.args[0]
elif isinstance(node, FuncCallStmt):
	self.interpret(node.func_call, env)
elif isinstance(node, RetStmt):
	# Exception raise
	raise Return(self.interpret(node.value, env))

# Return exception
class Return(Exception):
    pass
```
#### Compiler-Compilers
> 컴파일러를 만드는 컴파일러라는 의미.
> 지금까지 만들어 왔던 일련의 과정들을 BNF 문법을 정의한 파일을 통해 lexer 및 parser를 생성하는 도구이다.
> 대표적으로 YACC, Bison, ANTLR, etc가 있다.
> 대부분 사람들은 직접 컴파일러를 만들기보다 이러한 툴을 사용하여 언어를 개발한다.
#### Virtual Machine
> 지금까지 ast를 해석하고 해당 코드를 interperter를 통해 python으로 직접 실행했지만, 이제는 compiler를 통해 instructions(여전히 cpu에 고유한 instruction은 아님) set을 생성하고 이를 virtual machine을 통해 실행한다.
###### 특징
stack-based로 구현을 목표로 한다.
**Stack-based**
2 + 3 * 4
instruction set : PUSH 2,  PUSH 3,  PUSH 4, MUL, ADD
1. PUSH 2 => stack  = [2]
2. PUSH 3 => stack = [2, 3]
3. PUSH 4 => stack = [2, 3, 4]
4. MUL => stack = [2, 12]
5. ADD => stack = [14]
cf) 두 수의 연산 순서는 스택에 먼저 들어온 순서대로 연산한다. 예를 들어 2, 3을 가진 스택에 SUB를 하면 2 - 3이 된다.
###### Memory model
![img](https://study.com/cimages/multimages/16/1724cf83-a8ad-4ad5-aeca-0311114a819c_memory_alloc_cpp.png)
- Heap : 동적 메모리 할당
- Stack : 지역 변수, 함수 호출, etc
- Data : 전역 변수, static 변수, etc
- Code : 프로그램 코드
###### Instruction set
```
data instruction
PUSH <value> : stack에 value 추가
POP : stack에서 pop
ADD : stack에서 두개 pop 후 더한 값 push
SUB : stack에서 두개 pop 후 뺀 값 push
MUL : stack에서 두개 pop 후 곱한 값 push
DIV : stack에서 두개 pop 후 나눈 값 push
etc ...

control-flow instruction
JMP <label> : label로 이동
JMPZ <label> : stack에서 pop 후 0이면 label로 이동
JSR <label> : subroutine call
RTS : return from subroutine
HALT : 프로그램 종료
etc ...
```
###### Compiler
> ast를 통해 instruction set을 생성한다.
```python
class Compiler:
    def __init__(self):
        self.code = []

    def emit(self, instruction):
        self.code.append(instruction)

    def compile(self, node):
        if isinstance(node, Integer):
            value = (TYPE_NUMBER, float(node.value))
            self.emit(('PUSH', value))
        elif isinstance(node, Float):
            value = (TYPE_NUMBER, float(node.value))
            self.emit(('PUSH', value))
        elif isinstance(node, Bool):
            value = (TYPE_BOOL, node.value)
            self.emit(('PUSH', value))
        elif isinstance(node, String):
            value = (TYPE_STRING, stringify(node.value))
            self.emit(('PUSH', value))
        # etc ...

    def generate_code(self, root):
        self.emit(('LABEL', 'START'))
        self.compile(root)
        self.emit(('HALT', None))
        return self.code
```

**Global variable**
compiler에서 symbol table을 통해 global 변수를 추적하고 instruction set을 생성한다.
```python
class Symbol:
    def __init__(self, name):
        self.name = name

elif isinstance(node, Assignment):
	self.compile(node.right)
	symbol = self.get_symbol(node.left.name)
	if not symbol:
		# 현재 symbol이 없다면 새로 추가
		new_symbol = Symbol(node.left.name)
		self.globals.append(new_symbol)
		self.emit(('STORE_GLOBAL', new_symbol.name))
		self.num_globals += 1
	else:
		# 이미 존재하는 symbol이라면 update
		self.emit(('STORE_GLOBAL', symbol.name))
elif isinstance(node, Identifier):
	symbol = self.get_symbol(node.name)
	if not symbol:
		compile_error(f'Variable {node.name} is not defined', node.line)
	else:
		# global 변수에 접근
		self.emit(('LOAD_GLOBAL', symbol.name))
```

위 코드를 통한 instruction set 생성 예시
```
// test.pinky
x := 200
y := 300
z := x + y

// 컴파일러를 거친 instruction set
PUSH 200
STORE_GLOBAL x
PUSH 300
STORE_GLOBAL y
LOAD_GLOBAL x
LOAD_GLOBAL y
ADD
STORE_GLOBAL z
```

**Local variable**
compiler에서 scope_depth를 통해 local 변수를 추적하고 instruction set을 생성한다.
```python
# 기존 Symbol 클래스에 depth 추가
class Symbol:
    def __init__(self, name, depth = 0):
        self.name = name
        self.depth = depth

scopre_depth = 0

# 기존 조건문에 depth에 따라 STORE_LOCAL, LOAD_LOCAL instruction을 생성 코드 추가
elif isinstance(node, Assignment):
	self.compile(node.right)
	symbol, idx = self.get_symbol(node.left.name)
	if not symbol:
		new_symbol = Symbol(node.left.name, self.scope_depth)
		if self.scope_depth == 0:
			self.globals.append(new_symbol)
			self.emit(('STORE_GLOBAL', new_symbol.name))
			self.num_globals += 1
		else:
			self.locals.append(new_symbol)
			#vm의 stack에 이미 값이 존재하므로 STORE_LOCAL을 사용 필요하지 않음.
			#self.emit(('STORE_LOCAL', self.num_locals))
			self.num_locals += 1
	else:
		if symbol.depth == 0:
			self.emit(('STORE_GLOBAL', symbol.name))
		else:
			#local 변수는 vm stack의 위치를 통해 접근
			self.emit(('STORE_LOCAL', idx))
elif isinstance(node, Identifier):
	symbol, idx = self.get_symbol(node.name)
	if not symbol:
		compile_error(f'Variable {node.name} is not defined', node.line)
	else:
		if symbol.depth == 0:
			self.emit(('LOAD_GLOBAL', symbol.name))
		else:
			#local 변수는 vm stack의 위치를 통해 접근
			self.emit(('LOAD_LOCAL', idx))
```
###### Virtual Machine
```python
class VM:
    def __init__(self):
        self.stack = []
        self.pc = 0
        self.sp = 0
        self.is_running = False

    def run(self, instructions):
        self.is_running = True
        while self.is_running:
	        # args는 * operator를 통해 unpacking
            opcode, *args = instructions[self.pc]
			self.pc += 1
			# opcode와 같은 이름의 메소드 수행
            getattr(self, opcode)(*args)

    def HALT(self):
        self.is_running = False

    def PUSH(self, value):
        if len(self.stack) <= self.sp: self.stack.append(value)
        else: self.stack[self.sp] = value
        self.sp += 1
    
    def POP(self):
        self.sp -= 1
        return self.stack[self.sp]

	# etc ...
```