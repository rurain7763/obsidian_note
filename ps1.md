#### CPU
###### MIPS
> MIPS(Microprocessor without Interlocked Pipeline Stages)란 MIPS Technologies에서 개발한 RISC(CPU의 명령어셋)계열의 명령어 집합 체계이다.
> RISC는 컴퓨터 아키텍처 설계 철학 또는 개념의 한 종류이고, MIPS는 RISC를 기반으로 설계된 프로세서.
###### RISC의 주요 특징
- **단순한 명령어:** 명령어의 종류와 형식을 최소화하여 디코딩 및 실행 과정을 단순화합니다.
- **고정된 명령어 길이:** 모든 명령어의 길이를 동일하게 하여 명령어 가져오기(fetch) 단계를 효율적으로 처리합니다.
- **레지스터 기반 명령어:** 메모리 접근을 최소화하고, 레지스터 간 연산을 주로 사용하여 실행 속도를 높입니다.
- **적은 어드레싱 모드:** 메모리 주소 지정 방식을 단순화하여 명령어 디코딩 과정을 간소화합니다.
- **파이프라이닝:** 명령어 처리 단계를 여러 단계로 나누어 병렬적으로 처리하여 처리량을 증가시킵니다.
###### Coprocessor
1. cp0 : system control
2. cp2 : GTE(3d trasformation & matrix math)
3. MDEC : media decoder
###### Spec
1. 32개의 registers(각 32bit 씩 저장 가능)
2. 2 multiple & division registers
3. 32 bit data bus
4. 32 bit address bus
5. 1 ALU
6. 5 stage pipeline
###### Registers
[영상](https://courses.pikuma.com/courses/take/ps1-programming-mips-assembly-language/lessons/54214299-cpu-registers-load-instructions)
###### Memory map
[영상](https://courses.pikuma.com/courses/take/ps1-programming-mips-assembly-language/lessons/54214233-memory-map)
###### 5 Stage Pipeline
> 명령어를 처리하는 과정을 단계별로 나누어 효율적으로 처리하는 방식
> 명령어를 한줄 처리하고 다음 명령어를 순차적으로 처리하는 것이 아니라 여러 명령어를 동시에 단계별로 처리
![img|600](https://www.researchgate.net/publication/320662812/figure/fig1/AS:1137738267009024@1648269349168/Five-Stage-Pipelining.png)
- **IF (Instruction Fetch)**: 명령어를 메모리에서 가져옴
- **ID (Instruction Decode)**: 명령어를 해석하고 필요한 자원 확인
- **EX (Execute)**: 연산 수행
- **MEM (Memory Access)**: 메모리에 접근하는 연산 수행
- **WB (Write Back)**: 결과를 레지스터에 기록
#### Memory
32bit address - 1bytes
MMU(Memory Management Unit)가 없음. 따라서 가성 메모리도 없음.
Little endian
#### Size of MIPS types
- byte : 1byte
- half : 2byte
- word : 4byte
- dword : 8byte
#### Mips assembly
모든 명령어의 크기는 32bit 고정
###### Load instructions
`li $t0, 0xFF`
- Load immediate
- const 값을 특정 레지스터리에 저장
- 32bit 값까지 저장 가능
`la $t2, (특정 주소)`
- Load address
- 주솟값을 특정 레지스터리에 저장
`lb $t0, (메모리 주소 ex. 0x8000($t0))`
- Load 1byte from memory address
- 해당 메모리 주솟값의 1byte 값을 가지고와 특정 레지스터리에 저장
- `lh(2bytes), lw(5bytes)`도 있음
`lbu $t0, (메모리 주소)`
- `lb`명령어와 같지만 unsigned로 값을 읽어서 저장해라는 의미
`lui $t0, 0x8000`
- Load upper immediate
- 레지스터리의 upper bit(상위 16비트)에 const값을 저장
###### Store instructions
`sw $t0, (메모리 주소 ex. 0x8000($t0))`
- Store 4bytes to memory address
- 해당 레지스터리의 4bytes 값을 해당 메모리 주소에 저장
- `sh(2bytes), sb(1byte)`도 있음
###### Add & Subtract instructions
`add $t0, $t1, $t2`
- t0 = t1 + t2
`sub $t0, $t1, $t2`
- t0 = t1 - t2
`addi $t0, $t1, 5`
- t0 = t1 - 5
- `subi`는 없음 (`addi $t0, $t1, -5`로 가능함)
`addu $t0, $t1, $t2`
- unsigned add
`addui $t0, $t1, 5`
- unsigned add immediate
###### Jump & Branch instructions
`j (특정 라벨)`
- Uncoditional jump
- 특정 라벨로 점프
`jr $t0`
- Jump to register
- 레지스터리에 저장되어있는 주솟값으로 점프
`jal (특정 라벨)`
- Jump and link
- 현재 주소값을 $ra 레지스터리에 저장하고 해당 라벨로 점프
`beq $t0, $t1, (특정 라벨)`
- Branch equal
- `$t0`, `$t1`의 값이 같으면 해당 라벨로 점프
- `bnq(not equal), blt(less than), bgt(greater than), ble(less equal), bge(greater equal)`도 있음

cf) [그 외 명령어](https://import.cdn.thinkific.com/167815/XNWscWreSleMVc0BsPe8_MIPSCheatSheet.png)
###### 추가로 알아야 할 내용
bit shifting은 왜 사용할까?
비트 마스크로 활용할 수 있다
2의 승수로 곱하고 나누기를 빠르게 수행할 수 있다

모든 명령어의 크기는 32bit 고정인데 `li $t0, 0x12345678`과 같은 명령어는 어떻게 가능할까?
pseudo-op(기계어로 바로 번역되지 않고 어셈블리 단계에서 어셈블러가 여러 명령어로 대체)라는 기능으로 가능함

nop는 무엇이고 왜 사용할까?
nop는 no operation으로 아무것도 하지말라는 의미이며 5 stage pipline 설계에 의해 명령어가 실제 수행되는 단계가 비동기적으로 일어남
예로 nop가 없으면 아래와 같은 일이 발생
``` arm-asm
	lw $t0, 0x8000($t2)
	lw $t1, 0x8000($t3)
	ble $t0, $t1, TARGET ; ble 명령어가 실행되는 타이밍에 이미 아래의 명령어가 파이프라인을 타고 있음.
	lw $t0, 0x8000($t2) 
	lw $t1, 0x8000($t3) 
TARGET:
	; ...
```
또는 
``` arm-asm
	lw $t0, 0x8000($t2)
	lw $t1, 0x8000($t3)
	add $t0, $t1, $t2 ; 아직 위의 명령어가 실행되지 않는 상태에서 더하기를 진행하려고 함. (메모리에서 값을 가져올 때만 발생)
	lw $t0, 0x8000($t2) 
	lw $t1, 0x8000($t3) 
```
###### exercise
``` arm-asm
.psx
.create "factorial.bin", 0x80010000
.org 0x80010000

Main:
  li $a0, 5
  jal Factorial
  nop

InfinityLoop:
  j InfinityLoop
  nop
  
End:

Factorial:
  li $t1, 1
  li $t2, 0
  li $t3, 1
  li $t4, 1

OutterLoop:
  bgt $t1, $a0, OutterLoopEnd
  nop
  li $t4, 0
  li $t2, 0
InnerLoop:
  bge $t2, $t1, InnerLoopEnd
  nop
  add $t4, $t4, $t3
  addi $t2, $t2, 1
  b InnerLoop
  nop
InnerLoopEnd:
  move $t3, $t4
  addi $t1, $t1, 1
  b OutterLoop
  nop
OutterLoopEnd:

  move $v0, $t4
  jr $ra
  
.close
```
#### Negative number
제일 왼쪽 비트를 sign flag로 사용
보수법 사용
###### 이유
1. 0의 표현이 한가지만 존재한다
2. 더하기, 빼기 연산 법칙을 해치지 않는다
###### 예시
1byte의 경우 -128 + ... 으로 계산하면 됨
모든 비트를 반전시킨뒤 +1을 하면 됨
###### 추가로 알아야 할 내용
sign flag를 가진 바이트를 확장하면 어떻게 될까?
ex) 1byte -> 2bytes
0xb0101 => 0xb00000101 (아래 비트를 그대로 복사 후 나머지를 0로 채움)
0xb1101 => 0xb1111111101 (아래 비트를 그대로 복사 후 나머지를 1로 채움)

sign flag를 가진 바이트를 bit shifting 하면 어떻게 될까?
`sll, sllv, srl, srlv, rol, ror` :  부호 비트를 상관하지 않고 무조건적으로 이동 시키기 때문에 부호 비트를 잃는다
`sla, slav, sra, srav` : 부호 비트는 그대로 두고 이동 시키기 때문에 부호 비트를 보존한다
#### GPU
총 1024KB(512KB * 2)크기의 VRAM
1024 x 512 크기의 Frame buffer
cpu로 부터 32bit command를 받아 동작
15bit, 24bit color mode 지원
###### GP0
CPU가 Rendering 및 VRAM access를 위해 접근하는 포트(레지스터리)
[commands 종류](https://problemkaputt.de/psxspx-gpu-render-polygon-commands.htm)

Command 형식
1byte(command) + 3bytes(arguments)
Vertex : 2bytes(Y) + 2bytes(X)
Vertex : 2bytes(Y) + 2bytes(X)
Vertex : 2bytes(Y) + 2bytes(X)
if quad = Vertex : 2bytes(Y) + 2bytes(X)

``` arm-asm
// draw flat-shaded triangle
li $t1, 0x2000FF00
sw $t1, GP0($t0)

// vertex 1
li $t1, 0x00EF0000
sw $t1, GP0($t0)

// vertex 2
li $t1, 0x000000A0
sw $t1, GP0($t0)

// vertex 3
li $t1, 0x00EF013F
sw $t1, GP0($t0)
```
###### GP1
Display setup을 위해 접근하는 포트(레지스터리)
[commands 종류](https://problemkaputt.de/psxspx-gpu-render-polygon-commands.htm)

Command 형식
1byte(command) + 3bytes(arguments)
#### DMA
>DMA 컨트롤러(Direct Memory Access Controller)는 CPU의 개입 없이 메모리와 주변 장치 간에 데이터를 직접 전송하는 하드웨어 장치입니다. CPU가 데이터 전송에 관여하지 않으므로 CPU는 다른 작업을 수행할 수 있게 되어 시스템 전체의 성능이 향상됩니다.
###### DMA 컨트롤러의 작동 방식
1. **DMA 요청:** 주변 장치가 DMA 컨트롤러에게 데이터 전송을 요청합니다.
2. **버스 제어권 획득:** DMA 컨트롤러는 CPU에게 버스 제어권을 요청하고, CPU가 이를 승인하면 버스 제어권을 획득합니다.
3. **데이터 전송:** DMA 컨트롤러는 메모리와 주변 장치 간에 데이터를 직접 전송합니다. 이 과정에서 CPU는 관여하지 않습니다.
4. **버스 제어권 반환:** 데이터 전송이 완료되면 DMA 컨트롤러는 CPU에게 버스 제어권을 반환합니다.
5. **인터럽트 발생 (선택적):** 데이터 전송이 완료되면 DMA 컨트롤러는 CPU에게 인터럽트를 발생시켜 전송 완료를 알릴 수 있습니다.
###### DMA 컨트롤러의 장점
- **CPU 오버헤드 감소:** CPU가 데이터 전송에 관여하지 않으므로 CPU의 부담을 줄여줍니다.
- **데이터 전송 속도 향상:** CPU를 거치지 않고 직접 데이터를 전송하므로 전송 속도가 향상됩니다.
- **시스템 성능 향상:** CPU가 다른 작업을 수행할 수 있게 되어 시스템 전체의 성능이 향상됩니다.
###### DMA 컨트롤러의 사용 예시
- **디스크 드라이브:** 하드 디스크나 SSD에서 데이터를 읽거나 쓸 때 DMA를 사용하여 CPU의 부담을 줄입니다.
- **그래픽 카드:** 그래픽 카드가 메모리에서 이미지 데이터를 읽어올 때 DMA를 사용합니다.
- **사운드 카드:** 사운드 카드가 메모리에서 오디오 데이터를 읽어올 때 DMA를 사용합니다.
- **네트워크 인터페이스 카드:** 네트워크 카드가 메모리에서 네트워크 패킷을 송수신할 때 DMA를 사용합니다.
#### MIPS ABI calling convension
> MIPS ABI (Application Binary Interface) 호출 규약은 MIPS 아키텍처에서 함수를 호출하고 반환할 때 사용되는 규칙들을 정의합니다. 이 규약은 함수 호출 시 인자 전달 방식, 레지스터 사용 규칙, 스택 프레임 구성, 반환 값 전달 방식 등을 포함합니다. ABI를 준수하면 서로 다른 컴파일러로 컴파일된 코드들이 호환될 수 있습니다.

- **인자 전달 (Argument Passing):**
    - 처음 4개의 인자는 `$a0`, `$a1`, `$a2`, `$a3` 레지스터에 순서대로 전달됩니다.
    - 5번째 인자부터는 스택에 저장됩니다.
    - 부동 소수점 인자는 `$f12`, `$f14` 등의 부동 소수점 레지스터를 사용하여 전달됩니다.
- **반환 값 전달 (Return Value):**
    - 정수 반환 값은 `$v0` 레지스터에 저장됩니다.
    - 부동 소수점 반환 값은 `$f0` 레지스터에 저장됩니다.
- **레지스터 사용 규약 (Register Usage):**
    - `$a0` - `$a3`: 인자 레지스터 (Argument Registers)
    - `$v0` - `$v1`: 반환 값 레지스터 (Return Value Registers)
    - `$t0` - `$t9`: 임시 레지스터 (Temporary Registers) - 호출된 함수가 저장할 필요가 없습니다.
    - `$s0` - `$s7`: 저장 레지스터 (Saved Registers) - 호출된 함수가 사용하기 전에 저장하고 복원해야 합니다.
    - `$sp`: 스택 포인터 (Stack Pointer)
    - `$fp`: 프레임 포인터 (Frame Pointer) - 선택적으로 사용 가능
    - `$ra`: 복귀 주소 (Return Address)
- **스택 프레임 (Stack Frame):**
    - 함수 호출 시 스택에 생성되는 영역으로, 지역 변수, 저장된 레지스터 값, 복귀 주소 등을 저장합니다.
    - `$sp` 레지스터는 스택 프레임의 최상단 주소를 가리킵니다.
    - `$fp` 레지스터는 스택 프레임의 시작 주소를 가리키며, 지역 변수와 인자에 대한 고정된 오프셋을 제공합니다. (선택적 사용)
- **함수 호출 (Function Call):**
    - `jal` (jump and link) 명령어를 사용하여 함수를 호출합니다.
    - `jal` 명령어는 복귀 주소를 `$ra` 레지스터에 저장하고, 지정된 주소로 점프합니다.
- **함수 반환 (Function Return):**
    - `jr $ra` 명령어를 사용하여 호출한 함수로 복귀합니다.
#### Stack
sp(stack pointer) 레지스터리에 스택 메모리 시작 주속 기록 약속
###### 필요성
`DrawFlatTriangle`에 대한 subroutine 구현
``` arm-asm
  lui $a0, IO_BASE_ADDR
  li $s0, 0xFFFF00
  li $s1, 200
  li $s2, 40
  li $s3, 288
  li $s4, 56
  li $s5, 224
  li $s6, 200
  jal DrawFlatTriangle ; subroutine 호출!
  nop  

; =====================
; Args : 
; $a0 = IO_BASE_ADDR
; $s0 = Color
; $s1 = x1
; $s2 = y1
; $s3 = x2
; $s4 = y2
; $s5 = x3
; $s6 = y3
; =====================
DrawFlatTriangle:
  lui $t0, 0x2000
  or $t1, $t0, $s0
  sw $t1, GP0($a0)

  sll $s2, $s2, 16
  andi $s1, $s1, 0x0000FFFF
  or $t1, $s2, $s1
  sw $t1, GP0($a0)

  sll $s4, $s4, 16
  andi $s3, $s3, 0x0000FFFF
  or $t1, $s3, $s4
  sw $t1, GP0($a0)

  sll $s6, $s6, 16
  andi $s5, $s5, 0x0000FFFF
  or $t1, $s5, $s6
  sw $t1, GP0($a0)

  jr $ra
  nop
```
위 subroutine의 구현은 arguments가 많아 질 수록 더 많은 레지스터리를 필요로 하기에 한계가 있다는 점이 문제. 따라서 스택과 같은 개념이 필요함

```arm-asm
  la $sp, 0x00103DF0 ; 초기 스택 포인터

  lui $a0, IO_BASE_ADDR

  addiu $sp, $sp, -(4 * 7) ; 스택 포인터를 필요한 크기 만큼 할당

  ; 이하 offset을 이용하여 값 저장
  li $t0, 0xFFFF00
  sw $t0, 0($sp)

  li $t0, 200
  sw $t0, 4($sp)

  li $t0, 40
  sw $t0, 8($sp)

  li $t0, 288
  sw $t0, 12($sp)

  li $t0, 56
  sw $t0, 16($sp)

  li $t0, 224
  sw $t0, 20($sp)

  li $t0, 200
  sw $t0, 24($sp)

  jal DrawFlatTriangle ; subroutine 호출!
  nop  

; =====================
; Args : 
; $sp + 0 = Color
; $sp + 4 = x1
; $sp + 8 = y1
; $sp + 12 = x2
; $sp + 16 = y2
; $sp + 20 = x3
; $sp + 24 = y3
; =====================
DrawFlatTriangle:
  ; 이하 offset을 이용하여 값 가져오기
  lui $t0, 0x2000
  lw $t1, 0($sp)
  nop
  or $t2, $t0, $t1
  sw $t2, GP0($a0)

  lw $t0, 8($sp)
  lw $t1, 4($sp)
  nop
  sll $t0, $t0, 16
  andi $t1, $t1, 0x0000FFFF
  or $t2, $t0, $t1
  sw $t2, GP0($a0)

  lw $t0, 16($sp)
  lw $t1, 12($sp)
  nop
  sll $t0, $t0, 16
  andi $t1, $t1, 0x0000FFFF
  or $t2, $t0, $t1
  sw $t2, GP0($a0)

  lw $t0, 24($sp)
  lw $t1, 20($sp)
  nop
  sll $t0, $t0, 16
  andi $t1, $t1, 0x0000FFFF
  or $t2, $t0, $t1
  sw $t2, GP0($a0)

  addiu $sp, $sp, (4 * 7) ; 할당 한 크기만큼 스택 포인터 롤백

  jr $ra
  nop
```
#### Memory Allignment
>데이터의 시작 주소가 데이터 타입의 크기의 배수가 되도록 합니다. 예를 들어 4바이트 정수는 4바이트 경계에, 8바이트 배정밀도 부동소수점 수는 8바이트 경계에 정렬됩니다. 2바이트 자료형은 2바이트 경계에 정렬됩니다.

**메모리 정렬의 예:**
4바이트 정수(int)를 저장할 때, 다음과 같은 주소에 저장할 수 있습니다.
- **정렬된 경우:** 0x1000, 0x1004, 0x1008, ... (4의 배수)
- **정렬되지 않은 경우:** 0x1001, 0x1002, 0x1003, ...

만약 프로세서가 4바이트 단위로 메모리에 접근한다면, 정렬되지 않은 주소 0x1001에 저장된 4바이트 정수를 읽기 위해서는 0x1000부터 4바이트를 읽고, 0x1004부터 4바이트를 읽은 후 필요한 부분만 추출해야 합니다. 반면 정렬된 주소 0x1000에 저장된 정수는 한 번에 읽어올 수 있습니다.
###### global variable
```c
char char_var = 0;
short short_var = 0;
int int_var = 0;
long long_var = 0;

int main() {
	int a = int_var;
}
```
위 c 코드는 아래와 같이 변환됨
``` arm-asm
.org 0x80010000

char_var: .byte 0 ; 1byte
short_var: .hword 0 ; 2bytes
int_var: .word 0 ; 4bytes
long_var: .word 0 ; 4bytes

Main:
	la $t0, char_var
	lw $t1, 0($t0) 
	
.close
```
메모리 정렬의 규칙에 따라 전역 변수의 메모리 시작 주소의 경우는 아래와 같음 
- 0x8001000 : char_var => 1의 배수가 시작주소여야 함
- 0x8001002 : shor_var => 2의 배수가 시작주소여야 함
- 0x8001004 : int_var => 4의 배수가 시작주소여야 함
- 0x8001008 : long_var => 4의 배수가 시작주소여야 함

```c
int arr1[3] = {1, 2, 3};
int arr2[256];

int main() {
	int a = arr2[5];
}
```
위 c 코드는 아래와 같이 변환됨
``` mips-asm
.org 0x80010000

arr1: .word 1, 2, 3 ; 정렬할 수 있는 힌트가 있기 때문에 .align이 필요하지 않음
	  .align 2 ; 후에 오는 .space로 해당 크기만큼만 할당하는 배열 따라서 정렬 힌트가 없음. .align 필요 (.align 2^n을 표현 현재는 2^2 즉 4)
arr2: .space 256 * 4 ; 4의 배수의 시작주소로서 정렬됨

Main:
	la $t0, arr2
	lw $t1, 20($t0) ; arr2의 시작주소를 시작으로 offset을 이용하여 값을 찾음

.close
```
#### Psy-Q
> Psy-Q는 PS1용 게임 개발 툴킷으로, Psygnosis사에서 개발하였습니다. Psy-Q는 PS1용 게임을 개발하기 위한 컴파일러, 링커, 디버거, 에뮬레이터 등을 제공하며, PS1의 하드웨어를 직접 제어할 수 있는 라이브러리도 포함하고 있습니다.

cf) 환경 세팅 : https://github.com/NDR008/VSCodePSX
###### 기본 프레임
``` c
#include <stdlib.h>
#include <libgte.h>
#include <libetc.h>
#include <libgpu.h>

#define VIDEO_MODE 0
#define SCREEN_RES_X 320
#define SCREEN_RES_Y 240
#define SCREEN_CENTER_X (SCREEN_RES_X >> 1)
#define SCREEN_CENTER_Y (SCREEN_RES_Y >> 1)
#define SCREEN_Z 400

// 더블 버퍼링 세팅값을 가지는 구조체
typedef struct {
    DRAWENV draw[2];
    DISPENV disp[2];
} DoubleBuffer;

DoubleBuffer dbuff;
u_short currBuff;

void ScreenInit(void) {
    // GPU 초기화
    ResetGraph(0);

    // 첫번째 버퍼 설정값 세팅. 0,0,320,240가 전면 버퍼, 0,240,320,240이 후면 버퍼
    SetDefDispEnv(&dbuff.disp[0], 0,            0, SCREEN_RES_X, SCREEN_RES_Y);
    SetDefDrawEnv(&dbuff.draw[0], 0, SCREEN_RES_Y, SCREEN_RES_X, SCREEN_RES_Y);
  
    // 두번째 버퍼 설정값 세팅. 0,240,320,240가 전면 버퍼, 0,0,320,240이 후면 버퍼
    SetDefDispEnv(&dbuff.disp[1], 0, SCREEN_RES_Y, SCREEN_RES_X, SCREEN_RES_Y);
    SetDefDrawEnv(&dbuff.draw[1], 0,            0, SCREEN_RES_X, SCREEN_RES_Y);

    dbuff.draw[0].isbg = 1;
    dbuff.draw[1].isbg = 1;

    // 배경색 설정
    setRGB0(&dbuff.draw[0], 63, 0, 127);
    setRGB0(&dbuff.draw[1], 63, 0, 127);

    // 초기 버퍼 설정값을 가지고 있는 변수 세팅
    currBuff = 0;
    PutDispEnv(&dbuff.disp[currBuff]);
    PutDrawEnv(&dbuff.draw[currBuff]);

    // Geometry 초기화, 모든 오브젝트를 화면 중앙에 위치시키고, 화면 중앙을 기준으로 설정
    // 모든 좌표에 화면 중앙값을 더해준다고 생각하면 됨.
    InitGeom();
    SetGeomOffset(SCREEN_CENTER_X, SCREEN_CENTER_Y);
    SetGeomScreen(SCREEN_Z);
  
    // 디스플레이 온
    SetDispMask(1);
}

void DisplayFrame(void) {
    DrawSync(0);
    VSync(0);

    PutDispEnv(&dbuff.disp[currBuff]);
    PutDrawEnv(&dbuff.draw[currBuff]);

    currBuff = !currBuff;
}

void Setup(void) {
    ScreenInit();
}

void Update(void) {
}

void Render(void) {
    DisplayFrame();
}

int main(void) {
    Setup();
    while(1) {
        Update();
        Render();
    }
    return 0;
}
```
###### Primitive type 그리기
**Ordering table**
```c
#define OT_LENGTH 16

// 그려야 하는 object의 연결 리스트
u_long ot[2][OT_LENGTH];

// primitive object의 데이터를 저장하는 buffer
char primBuff[2][2048];
char* nextPrim;
```
cf)[참고](https://courses.pikuma.com/courses/take/ps1-programming-mips-assembly-language/texts/54230293-more-about-ordering-tables)

**그리기**
```c
ClearOTagR(ot[currBuff], OT_LENGTH);

// 1. 사각형 그림(Flat)
tile = (TILE*)nextPrim;
setTile(tile);
setXY0(tile, 82, 32);
setWH(tile, 64, 64);
setRGB0(tile, 0, 255, 0);
addPrim(ot[currBuff], tile);
nextPrim += sizeof(TILE);

// 2. 삼각형 그림(Flat)
triangle = (POLY_F3*)nextPrim;
setPolyF3(triangle);
setXY3(triangle, 64, 100, 200, 150, 50, 220);
setRGB0(triangle, 255, 0, 255);
addPrim(ot[currBuff], triangle);
nextPrim += sizeof(POLY_F3);

// 3. 사각형 그림(Gouraud)
quad = (POLY_G4*)nextPrim;
setPolyG4(quad);
setXY4(quad, 200, 100, 200, 200, 100, 100, 100, 200); // 순서에 신경 쓸 것! (0, 1, 2, 1, 2, 3) 순서로 그림
setRGB0(quad, 0, 0, 255);
setRGB1(quad, 255, 0, 0);
setRGB2(quad, 0, 255, 0);
setRGB3(quad, 255, 255, 255);
addPrim(ot[currBuff], quad);
nextPrim += sizeof(POLY_G4);

// ...

DrawOTag(ot[currBuff] + OT_LENGTH - 1); // 반전된 순서로 그림
```
#### GTE
> GTE (Geometry Transformation Engine)는 PS1의 CPU Coprocessor로서, 3D 그래픽 처리를 위한 하드웨어 가속기입니다. GTE는 3D 변환, 투영, 클리핑, 광원 계산 등을 하드웨어로 처리하여 CPU의 부담을 줄여줍니다.
###### Cube 그리기
```c
SVECTOR vertices[] = {
    { -128, -128, -128 },
    {  128, -128, -128 },
    { 128, -128, 128 },
    { -128, -128, 128 },
    { -128, 128, -128 },
    {  128, 128, -128 },
    { 128, 128, 128 },
    { -128, 128, 128 }
};

u_short faces[] = {
    0, 3, 2,
    0, 2, 1,
    4, 0, 1,
    4, 1, 5,
    7, 4, 5,
    7, 5, 6,
    5, 1, 2,
    5, 2, 6,
    2, 3, 7,
    2, 7, 6,
    0, 4, 7,
    0, 7, 3 
};

SVECTOR rotation = { 0, 0, 0 }; // 0 ~ 4096이 0 ~ 360도를 의미
VECTOR translation = { 0, 0, 900 };
VECTOR scale = { ONE, ONE, ONE }; // ONE은 1 << 12 (4096)한 값이다 (fixed-point number를 사용하기 때문)
)

MATRIX worldMatrix;
	
void Update(void) {
	// rotation, translation, scale matrix 구하기
	RotMatrix(&rotation, &worldMatrix);
	TransMatrix(&worldMatrix, &translation);
	ScaleMatrix(&worldMatrix, &scale);

	SetRotMatrix(&worldMatrix); // gte registery에 rotation matrix 등록
	SetTransMatrix(&worldMatrix); // gte registery에 translation matrix 등록
	
	// 각 face를 순회하면서 삼각형 그리기
	for(i = 0; i < NUM_FACES * 3; i += 3) {
		poly = (POLY_G3*)nextPrim;
		setPolyG3(poly);
		setRGB0(poly, 0, 255, 255);
		setRGB1(poly, 255, 0, 255);
		setRGB2(poly, 255, 255, 0);

#if NO_CULLING
		// depth buffer가 없으므로 z값을 평균내어 그림
		otz = 0;
		otz += RotTransPers(&vertices[faces[i + 0]], (long*)&poly->x0, &p, &flag);
		otz += RotTransPers(&vertices[faces[i + 1]], (long*)&poly->x1, &p, &flag);
		otz += RotTransPers(&vertices[faces[i + 2]], (long*)&poly->x2, &p, &flag);
		otz /= 3;
#elif CULLING
		// 함수의 반환값을 이용하여 culling (아마도 dot product값을 던져 주는듯?)
		nclip = RotAverageNclip3(
            &vertices[faces[i + 0]], 
            &vertices[faces[i + 1]], 
            &vertices[faces[i + 2]], 
            (long*)&poly->x0, 
            (long*)&poly->x1, 
            (long*)&poly->x2, 
            &p,
            &otz, 
            &flag);

		// 0보다 작으면 해당 삼각형 무시
        if(nclip <= 0) {
            continue;
        }
#endif
	
		if(otz > 0 && otz < OT_LENGTH) {
			addPrim(ot[currBuff][otz], poly);
			nextPrim += sizeof(POLY_G3);
		}
	}
}
```
#### Fixed-Point number
> ps1은 FPU(floating-Point Unit)가 없어 소수를 지원하지 않는다. 따라서 고정 소수점을 사용하여 소수를 표현한다. 고정 소수점은 정수부와 소수부로 나누어 표현한다. 예를 들어 16비트 고정 소수점은 8비트 정수부와 8비트 소수부로 나누어 표현한다.

cf) 반대 개념으로는 Floating-Point number가 있다. 현대 컴퓨터에서는 대부분 IEEE 754 표준을 따르는 32비트 또는 64비트의 부동 소수점을 사용한다.
###### 16.16 fixed-point number
16비트 정수부(부호 비트 포함)와 16비트 소수부로 나누어 표현한다. 예를 들어 1.0은 0x00010000으로 표현한다.
###### 20.12 fixed-point number
20비트 정수부(부호 비트 포함)와 12비트 소수부로 나누어 표현한다. 예를 들어 1.0은 0x00100000으로 표현한다.
cf) GTE는 20.12 fixed-point number를 사용한다. `long` type으로 표현한다.

**연산**
```c
#define ONE 4096

long num1 = 20 * ONE; // 20.0
long num2 = 30 * ONE; // 30.0

//addition
long add = num1 + num2; // 50.0

//subtraction
long sub = num1 - num2; // -10.0

//multiplication
long mul = (num1 * num2) >> 12; // 600.0

//division
#if V1
long div = (num1 << 12) / num2; // 0.6666666666666666
#else V2
long div = (num1 << ONE) /  num2; // 0.6666666666666666
#endif
```
#### Contoller input
> ps1의 컨트롤러는 CPU 메모리를 공유하지 않는 별도의 하드웨어로 구성되어 있습니다. 컨트롤러는 버튼, 스틱, 트리거 등을 포함하며, 버튼을 누르거나 스틱을 움직이면 컨트롤러가 패킷을 생성하여 컨트롤러 버스 통해 CPU에게 전달합니다. 패킷은 address + command + parameter로 구성되어 있습니다.
###### 패킷 구조
- address(1byte): 받아야 할 대상의 주소 ex) 0x01 :  1번 컨트롤러, 0x81 : 메모리 카드
- command + parameters: 명령
###### 코드
LIBETC.h 라이브러리를 사용하여 컨트롤러 입력을 받는다.
```c
u_long padState;

void Setup(void) {
	// 첫번째 컨트롤러 초기화
	PadInit(0);
}

void Update(void) {
	// 첫번째 째 컨트롤러 입력 상태 읽기
	padState = PadRead(0);

	if(padState & _PAD(0, PADLleft)) {
		// Do something
	}
}
```
#### CD-ROM
> sector가 기본 단위이며, 1 sector는 2048 bytes로 구성되어 있습니다. sector의 그룹은 track이라고 하며, track의 그룹은 session이라고 합니다.
> ISO-9660 파일 시스템을 사용하여 CD-ROM에 파일을 저장합니다. ISO-9660은 CD-ROM 파일 시스템의 표준으로, 파일 이름, 디렉터리 구조, 파일 속성 등을 정의합니다.
###### 저장 구조 예시
```
CD
├── Session 1
│   ├── Track 1
│   │   ├── Textures
│   │   │   ├── texture1.bmp
│   │   │   ├── texture2.bmp
│   │   ├── Models
│   │   │   ├── model1.obj
│   │   │   ├── model2.obj
```
###### make iso
32bit machine : [영상](https://courses.pikuma.com/courses/take/ps1-programming-mips-assembly-language/lessons/54232873-cd-rom-basics)
64bit machine : [영상](https://courses.pikuma.com/courses/take/ps1-programming-mips-assembly-language/lessons/54232971-generating-an-iso-on-windows-11)
###### file read
```c
char* FileRead(const char* filename, u_long* size) {
    CdlFILE file;
    int num_sectors;
    char* buffer = NULL;

    *size = 0;
    if(CdSearchFile(&file, (char*)filename)) {
        num_sectors = (file.size + (CD_SECTOR_SIZE - 1)) / CD_SECTOR_SIZE;

        buffer = (char*)malloc(num_sectors * CD_SECTOR_SIZE);
        if(!buffer) {
            printf("Failed to allocate memory for file: %s\n", filename);
            return buffer;
        }

        CdControl(CdlSetloc, (u_char*)&file.pos, 0); // Set the location of the file
        CdRead(num_sectors, (u_long*)buffer, CdlModeSpeed); // Read the file
        CdReadSync(0, 0); // wait for the read to finish

        *size = file.size;
    } else {
        printf("File not found: %s\n", filename);
    }

    return buffer;
}
```

**파일 read 이후 모델링 데이터 불러오기 예시**
```c
char* bytes;
u_long size;
bytes = FileRead("\\MODEL.BIN;1", &size);
    
if(bytes) {
	u_long bytesRead = 0;
	cube.numVertices = GetShortBE(bytes, &bytesRead);
	cube.vertices = (SVECTOR*)malloc3(cube.numVertices * sizeof(SVECTOR));
	for(short i = 0; i < cube.numVertices; i++) {
		cube.vertices[i].vx = GetShortBE(bytes, &bytesRead);
		cube.vertices[i].vy = GetShortBE(bytes, &bytesRead);
		cube.vertices[i].vz = GetShortBE(bytes, &bytesRead);
	}

	cube.numFaces = GetShortBE(bytes, &bytesRead) << 2;
	cube.faces = (short*)malloc3(cube.numFaces * sizeof(short));
	for(short i = 0; i < cube.numFaces; i++) {
		cube.faces[i] = GetShortBE(bytes, &bytesRead);
	}

	cube.numColors = (short)GetByte(bytes, &bytesRead);
	cube.colors = (CVECTOR*)malloc3(cube.numColors * sizeof(CVECTOR));
	for(char i = 0; i < cube.numColors; i++) {
		cube.colors[i].r = (short)GetByte(bytes, &bytesRead);
		cube.colors[i].g = (short)GetByte(bytes, &bytesRead);
		cube.colors[i].b = (short)GetByte(bytes, &bytesRead);
		cube.colors[i].cd = (short)GetByte(bytes, &bytesRead);
	}

	free(bytes);
}
```
#### Textured polygon
texture를 vram에 로드해야 한다.
uv값은 left-top이 (0, 0)이고 right-bottom이 (texture_width, texture_height)이다.
uv값 말고도 TPAGE(texture page)와 CLUT(color look-up table)을 지정해야 한다.
###### TPAGE
하나의 page는 1024 x 512 크기의 frame buffer에 16 x 2 개의 cell로 나누어져 있는 것 중에 하나이다. (cell은 64 x 256 크기)
TPAGE는 frame buffer 상의 page의 시작 위치이고 그리고자 하는 texture의 offset 값이라고 생각하면 된다.
###### CLUT
CLUT는 color look-up table로서 texture의 색상을 지정한다.
###### TIM 파일
TIM 파일은 텍스처 이미지를 저장하는 파일이다. TIM 파일은 header와 이미지 데이터로 구성되어 있다.
```c
// libgpu.h의 TIM_IMAGE 구조체
typedef struct {
	u_long  mode;		/* pixel mode */
	RECT	*crect;		/* CLUT rectangle on frame buffer */
	u_long	*caddr;		/* CLUT address on main memory */
	RECT	*prect;		/* texture image rectangle on frame buffer */
	u_long	*paddr;		/* texture image address on main memory */
} TIM_IMAGE;
```
###### 코드
```c
u_long* bytes;
u_long size;

bytes = (u_long*)FileRead(file, &size);

if(bytes) {
	u_long bytesRead = 0;
	TIM_IMAGE tim;

	// TIM 파일을 읽어서 TIM_IMAGE 구조체에 저장
	OpenTIM(bytes);
	ReadTIM(&tim);

	// texture 데이터를 vram에 로드
	LoadImage(&tim.prect, tim.paddr);
	DrawSync(0);

	// clut 데이터를 vram에 로드
	if(tim.mode & 0x8) {
		LoadImage(&tim.crect, tim.caddr);
		DrawSync(0);
	}

	free(bytes);
}
```