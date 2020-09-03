---
title: buffer overflow attack
published: true
---

# buffer overflow attack?

>버퍼의 공간의 크기보다 큰 데이터를 저장하게 해서 일어나는 오버플로우(overflow)를 이용한 공격이다.

* * *

## 8086 메모리의 시스템 구조


![](./assets/buf1.png){: width="300" height="300"}
8086 시스템의 기본적인 메모리 구조

* 시스템이 초기화 되기 시작하면 시스템은 커널을 메모리에 적재시키고 가용 메모리 영역을 확인하게 된다.

>시스템 운영에 필요한 기본적인 명령어 집합을 커널에서 찾기 때문애 커널 영역은 반드시 저 위치에 있어야 한다.

```
메모리 영역에 주소를 할당할 수 있는 범위
32bit: 0 ~ 2^32-1
64bit: 0 ~ 2^64-1
```

* 운영체제는 프로세스를 segment라는 단위로 묶어서 가용 메모리 영역에 저장시킨다
* 오늘날의 시스템은 멀티 테스킹이 가능하므로 여러 개의 segment들이 저장될수있다.

* * *


![](./assets/buf2.png)

Segment는 그림과 같은 구조를 하고있다.
 * code segment

   >시스템이 알아들을 수 있는 명령어(instruction)이 들어 있다. 
   >instruction은 기계어 코드이고, 명령을 수행하면서 많은 분기 과정과 점프, 시스템 호출 등을 수행한다.
   >분기와 점프의 경우 메모리 상의 특정 위치에 있는 명령을 지정해 주어야 한다.

   * segment는 segment selector에 의해서 offset을 찾을 수 있고, 자신의 시작 위치로부터 위치에 있는 명령을 수행할 지를 결정하게 된다. 따라서 physical address는 offset + logical address라고 할 수 있다. 

* * *

 * data segment
   
   1. 프로그램이 실행시에 사용되는 데이터(전역 변수)가 들어간다.
   2. data segment는 다시 네 개의 data segment로 나뉜다.
      >data structure

      >데이터 모듈

      >동적 생성 데이터

      >공유 데이터 부분

* * *

 * stack segment

   1. 현재 수행되고 있는 handler, task, program이 저장하는 데이터 영역   
   2. 우리가 사용하는 버퍼가 자리잡게 된다.
   3. 프로그램이 사용하는 multiple 스텍을 생성할 수 있고, 스텍들간의 switch가 가능하다.
   4. 지역 변수들이 자리 잡는 공간이다.

```
스텍(stack)

스텍은 필요한 크기만큼 만들어지고 프로세스의 명령에 의해 데이터를 저장해 나가는 과정을 거치게 된다.
   -stack pointer(SP)라고 하는 레지스터가 스텍의 맨 꼭대기를 가리키고 있다. 
   -PUSH와 POP instruction에 의해서 데이터를 저장하고 읽어 들이는 과정을 수행한다.

   스텍은 나중에 들어간 데이터가 가장 먼저 나오는 LIFO의 구조이다. 

```
* * *

### 8086 CPU 레지스터 구조 

* 레지스터(register)
  -CPU가 흩어져 있는 명령어 집합과 데이터들을 적절하게 집어내고 읽고 저장하기 위해서는 여러 가지 공간이
   필요하다. 또한 재빨리 읽고 쓰기를 해야 하는 데이터들이므로 CPU 내부에 존재하는 메모리를 사용한다. 
   이러한 저장 공간을 레지스터(register)라고 한다.

*레지스터의 종류*
```

1. 범용 레지스터 : 피연산자와, 메모리 포인터가 저장되는 레지스터다.

   -프로그래머가 임의로 조작할 수 있게 허용되어 있다.
   
   1. EAX: 피연산자와 연산 결과의 저장소
   2. EBX: DS segment안의 데이터를 가리키는 포인터
   3. ECX: 문자열 처리나 루프를 위한 카운터
   4. EDX: I/O 포인터
   5. ESI: DS 레지스터가 가리키는 data segment 내의 어느 데이터를 가리키고 있는 포인터
   6. EDI: ES 레지스터가 가리키고 있는 data segment 내의 어느 데이터를 가리키고 있는 포인터
   7. ESP: SS 레지스터가 가리키는 stack segment의 맨 꼭대기를 가리키는 포인터
   8. EBP: SS 레지스터가 가리키는 스텍상의 한 데이터를 가리키는 포인터


2. 세그먼트 레지스터 : code segment, data segment, stack segment를 가리키는 주소가 들어 있다.

   -CS 레지스터: code segment를 가리킴
   -DS, ES, FS, GS 레지스터: data segment를 가리킴
   -SS 레지스터: stack segment를 가리킴

3. 플래그 레지스터 : 프로그램의 현재 상태나 조건 등을 검사하는데 사용되는 플래그들이 있다.

    -Status flags(상태 플래그)
      1.CF - carry flag
      2.PF - parity flag
      3.AF - adjust flag
      4.ZF - zero flag
      5.SF - sign flag
      6.OF - overflow flag

      DF - Direction flag

    -System flags(시스템 플래그)
      1.IF - interrupt enable flag
      2.TF - trap flag
      3.IOPL - I/O privelege level field
      4.NT - nested task flag
      5.RF - resume flag
      6.VM - virtual-8086 mode flag
      7.AC - alignment check flag 
      8.VIF - vertual interrupt flag
      9.VIP - virtual interrupt pending flag
      10.ID - identification flag

    -Instruction Pointer

4. 인스트럭션 포인터: 다음 수행해야 하는 명령이 있는 메모리 상의 주소가 들어가 있다.

```
* * *         

#### 프로그램 구동 시 Segment에서는 어떤 일이?

```
 void function(int a, int b, int c){

               char buffer1[15];
               char buffer2[10];
 }

 void main(){
               function(1,2,3);
 }

 ```

 위에처럼 간단한 프로그램을 만들고 컴파일을 해서 gdb로 이 프로그램을 어셈블리어로 보자.

 보고 대충 해석해보면 ebp를 넣어주고 esp를 ebp로 이동시켜주고 esp를 8byte 빼주고 and 연산을 해준뒤 

 eax에 0x0값을 넣어주고 esp에 값에서 eax에 값만큼 빼주고 esp를 4byte만큼 빼주고 

 인자값 1, 2, 3을 넣어주고 call를 사용해서 0x80482f4 <fundtion> 을 수행해주고 

 function()함수에서도 함수 프롤로그 수행해서 ebp, esp 위치를 잡아주고 esp를 40byte 빼준다.

 그리고 leave를 사용해서 함수 프롤로그 작업을 되돌리고 ret을 사용하여 main()함수 이전으로 돌아간다.

 이렇게 해석할 수 있다.

 ```
 추가 설명

 sub - 빼기
 mov - 이동
 push- 넣기 
 call - 해당 명령 수행하기
 and - and연산해주기
 
 그리고 function함수에서 esp를 40byte 빼주는 이유는 스텍은 16배수로 할당되기 때문에 buffer1에 16byte
 buffer2에 16byte가 할당되게 된다. 그리고 dummy값이 8byte 할당되기 때문에 총 40byte가 확장되게된다.

 ```

 * * *

 ##### buffer overflow의 이해 

    공격자는 버퍼가 넘칠 떄, 즉 버퍼에 데이터를 쓸 떄 원하는 코드를 넣을 수가 있다.

    이를 이용해서 return address에 원하는 코드를 위치해놓은 주소를 집어 넣어서 EIP에 공격자의 

    코드가 있는 곳의 주소가 들어가게해서 공격을 하는 방법이다.

    ![](./asstes/buf3.png){: width="250" height="200"}

    위에 코드를 이용해서 ("/bin/sh",...)를 수행하게 된다. 이것이 buffer overflow를 이용한 공격 방법이다.


   ```
   byte order 

        1. big endian : 바이트 순서가 낮은 메모리 주소에서 높은 메모리 주소로 된다.

        2. little endian : 높은 메모리 주소에서 낮은 메모리 주소로 된다.

   ```

   * 쉘 코드 
      
        -사용자의 키보드입력을 받아서 실행파일을 실행시키거나 커널에 어떠한 명령을 내릴 수 있는 대화통로

   * Dynamic Link Library 
      
        -운영체제가 많이 사용되는 함수들의 기계어 코드를 가지고 있고 다른 프로그램들이 이 기능을 
         빌려 쓸 수 있도록 해주는 것 

        -윈도우(DLL), 리눅스(libc)

   * Static Link Library
    
        -버전에 따라 호출 형태나 링크 형태가 달라질 수 있기 떄문에 영향을 받지 않기 위해 기계어 코드를
         직접 가지고 있게 할 수 있는 방법

   * 쉘을 띄우기 위한 과정
       
       1. 스텍에 execve()를 실행하기 위한 인자들을 제대로 배치하고
       2. NULL과 인자값의 포인터를 스텍에 넣어 두고
       3. 범용 레지스터에 이 값들의 위치를 지정해 준 다음에 
       4. interrupt 0x80을 호출하여 system call 12를 호출하게 하면 된다.

   >하지만 이 코드는 /bin/sh가 data segment에 저장되어 있기 때문에 이용할 수 있지만 
   >buffer overflow 공격 시점에서는 /bin/sh가 어느 지점에 저장되어 있는지 모르기 때문에 
   >직접 넣어주어야 한다.

   * NULL의 제거
     
        -push 0x0와 같은 어셈블리 코드는 기계어 코드로 6a 00 이다. 이것을 무자열 형태로 전달할떄
         char형 배열, 즉 문자열에서는 0의 값을 만나면 문자열의 끝으로 인식하게 되기때문에 
         0x00 뒤에 값을 무시해버린다 그래서 \x00인 기계어 코드를 생기지 않게 만들어줘야한다.

* * *

* 다른 방법 

   1. 쉘 코드를 저장할 변수를 int형으로 만들어주는 방법
      >littel endian 순서로 정렬해야 하며 int형이므로 4 byte 단위로 만들어줘야 한다. 

```

root권한을 얻을 수 있는 방법은 setuid 비트가 set되어 있는 프로그램을 이용 할 수 있다.

    -setuid 비트가 set되어  있는 프로그램을 오버플로우시켜 쉘 코드를 실행시키고 루트의
     쉘을 얻어낼 방법

     >setreuid 함수를 찾아 인터럽트를 호출하는 부분을 찾고 이것을 우리가 만든 
      쉘 코드 앞부분에 단순히 붙여 넣어주기만 하면 됨.

```



buffer overflow 

  * 고전적인 방법
    
     -쉘 코드 앞을 NOP로 채우고 return address를 NOP로 채워져 있는 영역 어딘가의 주소로 바꾸면 
      operation의 흐름은 NOP를 타고 쉘 코드가 있는 곳까지 들어 갈수 있게 되는것이다.

  * 환경변수를 이용하는 방법
    
     -환경 변수를 하나 만들고 이 안에 쉘 코드를 넣은 다음에 취약한 프로그램에 환경변수의 address를
      return address에 넣어줌으로써 쉘 코드를 실행하게 할 수 있다.

* * *

###### eggshell 기법 

환경변수를 만들어 환경 변수들의 크기가 늘어나게 되고 main함수의 base pointer는 그만큼 낮은 곳에 
자리잡게 된다.

이렇게 만든 egg라는 환경변수안에는 많은 NOP가 있음으로 해서 구해진 stack pointer가 쉘 코드의 
정확한 시작접을 가리키지 않더라도 instruction pointer가 흘러서 쉘 코드 시작접까지 
도달할 수 있게 되는 것이다.