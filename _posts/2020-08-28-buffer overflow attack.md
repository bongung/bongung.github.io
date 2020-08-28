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