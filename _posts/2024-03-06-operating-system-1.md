---
title: "[운영체제] 1. CPU의 Instruction 처리 Cycle에 대해서 알아보자"
date: 2024-03-06 23:00:00 +0900
categories: [Study, 운영체제]
tags: [study, operating system, instruction]     
# TAG names should always be lowercase
toc: true
---

## **Instruction이란?**

### **정의**

> **GPT**
>- **Instruction(명령어)은 컴퓨터에서 실행되는 작업을 나타내는 기본 단위입니다. 프로그램은 여러 개의 명령어로 구성되며, 컴퓨터는 이러한 명령어를 순차적으로 실행하여 원하는 작업을 수행합니다.**
> 
> 
>- **명령어는 컴퓨터가 이해할 수 있는 형태로 작성된 코드로, 일련의 비트로 표현됩니다. 명령어는 CPU가 해석하고 실행하는데, CPU는 명령어를 가져와 해독하고 해당하는 동작을 수행합니다.**
> 

Instruction의 정의는 위의 Chat GPT에서 자세하게 답변을 해주어 추가적인 설명을 하지 않겠다.

CPU의 주요 기능은 Instruction 처리이다. 이는 CPU 단독으로만으로 동작하지 않고 Computer System의 핵심 구성요소들과 함께 동작하며 처리하게 된다.

### **구성**

Instruction의 구성은 Opcode와 Operand로 나뉘게 된다.

Opcode는 명령어의 종류를 나타내며, Operand는 주소값을 나타낸다.

Opcode와 Operand가 합쳐져서 하나의 Instruction을 구성하게 되는데, 각각의 크기와 전체 크기는 시스템의 종류에 따라서 다양한 템플릿을 사용한다.

하지만 본 정리에서는 편의를 위해 4bit의 Opcode와 12bit의 Operand로 구성되어있는 총 16bit인 Instruction 템플릿을 사용하여 진행할 것이다.

![instructions](/assets/img/posts/2024-03-06/instruction.png)

### **Instruction Set Architecture (ISA)**

> **GPT**
>- **ISA(Instruction Set Architecture)는 컴퓨터 시스템에서 프로세서와 소프트웨어 간의 인터페이스를 정의하는 기술적인 명세입니다. ISA는 프로세서가 인식하고 처리할 수 있는 명령어 세트와 그 명령어들의 동작 방식을 정의합니다. 이를 통해 소프트웨어 개발자는 특정 ISA를 기반으로 소프트웨어를 개발하고, 프로세서는 해당 ISA를 따르는 명령어를 실행합니다.**

- SW와 HW 사이의 명령어가 정의된 집합 구조를 가지는 방법론이다.
- Operation이 주어졌을 때 해당하는 Instruction이 정의된다.
- ISA의 종류에는 x86, x86-64, IA64, ARM 등이 있다.

### **Instruction Decode and Excute**

Processor는 Instruction을 해석해서 아래와 같은 필요한 동작을 수행한다.

- Data Processing (+, -, and, or…)
- Control(Flow Control - call)
- Processor Memory(Load, Store)
- Processor-I/O

예를 들면 

0001은 memory에서 데이터를 Load하여 AC에 넣는 동작이고,

0010은 AC의 데이터를 memory에 Store하는 동작이고,

0101은 AC의 값에 memory의 데이터를 Add 하는 동작과 같이 정의될 수 있다.

## **Computer System의 핵심 구성**

### **구성 요소**

Instruction의 정의에서 언급했다시피 Instruction을 처리하기 위해서는 Computer System의 핵심 구성요소들이 함께 동작하여 처리하게 된다. Computer System의 핵심 구성은 아래와 같이 크게 4가지로 나눌 수 있다.

- **CPU** : 중앙 처리 장치
    - 컴퓨터에서 가장 중요한 구성 요소 중 하나이다.
    - 모든 작업을 제어하고 수행하며 프로그램에서 명령어를 해석, 실행하는 역할을 한다.
- **Memory** : 저장 장치
    - 주기억장치(Main Memory, RAM)과 보조기억장치(Secondaty Memory, HDD/SSD)으로 분류된다.
    - 주기억장치는 컴퓨터가 현재 사용하는 데이터를 저장하는 장치이다. 휘발성으로 전류 공급이 끊기면 데이터가 날아간다. 단, 데이터 접근 속도가 보조기억장치보다 눈에 띄게 빠르다.
    - 보조기억장치는 데이터를 반영구적으로 저장하는 장치이다. 비휘발성으로 전류 공급에 상관없이 데이터를 저장할 수 있다.
- **I/O** : 입출력 장치
    - 사용자와 컴퓨터 사이의 데이터 전달을 담당한다.
    - 키보드, 마우스, 모니터, 스피커
- **Bus** : 데이터 전송 통로
    - 내부에서 데이터나 신호를 전송하는 통로이다.

이렇게 4가지의 핵심 구성이 존재하며, Instruction의 Cycle에 대해서 정리하기 전에 위와 관련된 내용인 Processor, Memory, Register를 먼저 살펴보자.

### **Processor**

> **GPT**
>- **Processor는 컴퓨터 시스템에서 중앙 처리 장치(Central Processing Unit, CPU)를 의미합니다. CPU는 컴퓨터의 두뇌로서, 프로그램의 명령을 실행하고 데이터를 처리하는 역할을 담당합니다. 이는 컴퓨터의 핵심 구성 요소로, 프로그램의 실행 속도와 성능에 큰 영향을 미칩니다. Processor는 주파수, 아키텍처, 핵 수 등 다양한 요소로 구성되며, 성능은 이러한 요소들의 조합에 따라 결정됩니다. 최신 프로세서는 고성능 및 저전력을 위해 여러 기술과 최적화를 적용하여 제조됩니다.**
> 

Processor는 CPU의 다른 이름이라고도 할 수 있다. 지금은 Processor라고 언급했는데, 이는 CPU가 과거에는 어떻게 존재했고 현재에는 어떻게 발전되고 있는지를 간단하게 살펴보기 위해서이다.

1. Arithmetic Logic Unit(ALU, 산술 논리 장치)
    - 초기에는 CPU가 존재하지 않았고, ALU를 사용하여 수학 계산과 논리 연산을 수행하였다.
2. Central Processing Unit(CPU, 중앙 처리 장치)
    - ALU와 Control Unit, Register로 구성되어, Instruction Sets를 수행한다.
    - Frequency를 높여 성능을 향상시키려 하지만, 이에 비례하여 Power도 많이 요구되기 때문에 성능을 향상시키는데 한계가 존재한다.
3. Multiprocessors
    - 초기 CPU는 단일 코어로 존재하였고, 하나의 코어의 성능을 높이기 위해서 단일 칩 집적도를 향상시켰다.
    - 하지만, 한개의 칩의 집적도를 높이는데에는 명백한 한계가 존재하였고, 이제 단일에서 벗어나서 여러개의 코어가 들어간, Multiprocessor가 나오게 되었다.
4. Graphical Processing Units(GPU, 그래픽 처리 장치)
    - 점점 성능이 좋아지고, 이제 CPU만으로는 처리 능력에 한계가 다가오게 되었다.
    - 프로그램에서 산술적인 계산의 수가 점점 늘어나고, 수많은 산술 계산을 처리하기 위해 GPU를 개발, 사용하게 되었다. (ALU가 빽빽하게 들어가있다.)
    - 이는 반복적인 계산에 특화하여 병렬적으로 처리할 수 있도록 하는 장치이다.
    - GPU는 SIMD 기법을 사용해서 데이터 배열에 대한 효율적인 연산을 제공하게 된다.
5. System on a Chip(SoC)
    - 주로 저전력 소비 디바이스에 많이 사용된다.
    - CPU, GPU, RAM 등이 모두 한개의 칩에 한번에 들어간다.
    - CPU 또한 크기별로 Big, Middle, Little Core가 각각 들어가는데, 이는 Frequency가  높아짐에따라 Power 소모도 높아지는 것을 고려한 것으로 적은 부하가 요구되는 작업은 Little Core로, 큰 부하가 요구되는 작업은 Big Core로 수행하여 전력 소모를 최적화해 사용한다.
    
    ![M1 칩의 구조](/assets/img/posts/2024-03-06/m1chip.png){: width="200"}
    
    M1 칩의 구조
    

### ****폰 노이만 구조(Von Neumann Architecture)**

![폰 노이만 구조](/assets/img/posts/2024-03-06/von_neumann_architecture.png){: width="400"}

현재의 대부분의 범용 컴퓨터에서 가지는 컴퓨터 아키텍처이다.

해당 아키텍처는 CPU, Memory, Program으로 구성된다. 위처럼 CPU와 Memory는 분리되어있고, Bus를 통해서 명령어와 데이터의 읽고 쓰기를 진행한다.

명령어와 데이터를 같이 Bus를 통해 주고받기 때문에 명령어를 불러오고 있을 때 데이터를 동시에 불러올 수 없고 각각 따로 받아와야한다.

해당 아키텍처는 Bus를 통해 데이터를 불러오는데, 이 Bus를 통하면 CPU가 메모리에서 데이터를 느리게 불러오는 단점이 있다. CPU에서 처리하는 속도에 비하면 Data의 Transfer에 대한 Overhead가 크다는 것이다.

### **Memory**

> **GPT**
>- **Memory(메모리)는 컴퓨터 시스템에서 데이터와 명령어를 저장하고 읽고 쓰는 장치입니다. 메모리는 컴퓨터의 중요한 구성 요소로서, 프로그램 및 데이터의 일시적인 저장소 역할을 합니다. 컴퓨터가 실행 중인 프로그램은 메모리에 저장되어야만 CPU가 해당 명령어를 읽고 실행할 수 있습니다.**
> 

**Random Access Memory**

1. Static RAM → Cache
    - 초기부터 존재했던 메모리이다. 6개의 Transistor로 1bit의 데이터를 저장할 수 있다. (Flip-Flop, Latch으로 구성)
    - 엄청나게 빠른 속도(대략 2ns 이내)로 데이터 접근이 가능하며, DRAM의 단점인 Refresh가 필요없다.
    - 속도는 빠르지만, 6개의 Transistor로 구성되기 때문에 집적률이 낮고 가격이 비싸다.
2. Dynamic Ram → Main Memory
    - SRAM의 낮은 집적률을 극복하고자 개발된 메모리이다. 1개의 Transister와 1개의 Capacitor로 1bit의 데이터를 저장할 수 있고, SRAM에 비해 높은 집적도를 가질 수 있다.
    - SRAM에 비하면 느리지만, 그래도 엄청나게 빠른 속도(대략 15ns 이내)로 데이터 접근이 가능하다.
    - 데이터를 저장하기 위해 전하를 유지하는 Capacitor를 사용한다. 하지만 Capacitor는 시간이 지나면 전하가 누수되는데, 이는 데이터의 손실을 발생시킬 수 있어 주기적인 데이터 덮어씌우기가 필요하다. 이를 Dynamic Refresh라고 하며, DRAM에서는 해당 과정이 필수적이다.

이러한 Memory는 CPU와 GPU에 모두 존재한다. 

만일 Memory가 CPU나 GPU에 존재하지 않게 된다면, Bus로 연결된 다른 Memory에 접근해서 명령어나 데이터를 가져와야 하는데, 폰 노이만 구조에서 언급했다시피 Bus를 통하게 되면 처리하는 속도에 비해 Data의 Transfer에 대한 Overhead가 크기 때문에 이는 성능의 하락으로 이어질 수 있다.

따라서 Memory는 CPU와 GPU에 모두 존재하게 되고, Heterogeneous Memory 기술을 통해서 성능을 향상시킬 수 있다.

** **Heterogeneous Memory란**

> **GPT**
>- **Heterogeneous Memory(이하 HM)는 다양한 유형의 메모리를 하나의 시스템에서 통합하여 사용하는 기술을 의미합니다. HM은 주로 컴퓨터 시스템에서 메모리 계층 구조를 최적화하고 성능을 향상시키기 위해 사용됩니다.**
> 

여러 종류의 메모리를 통합해서 접근할 수 있게 하여, 성능을 향상시키는 방법이다.

프로그램을 실행하는 방법에는 크게 두가지가 존재한다.

- Load then Excute : GPU에서 CPU Memory에 접근하여 실행한다고 할 때 CPU Memory에서 해당 부분을 GPU로 옮기고 실행하는 방식이다. 반복적으로 사용되고, 재활용될 수 있는 Hot Data(접근 빈도가 높은 데이터)를 실행할 때 효율적이다.
- Direct host access : GPU에서 CPU Memory에 접근하여 실행한다고 할 때 CPU Memory에 GPU가 직접 접근해서 바로 실행하는 방식이다. 가끔 사용되는 Cold Data(접근 빈도가 낮은 데이터)를 실행할 때 효율적이다.

3. Hard Disk Drive(HDD)
    - 자기장을 사용하여 데이터를 저장하는 방식이다.
        
        ![hdd](/assets/img/posts/2024-03-06/hdd.png)
        
        [commons.wikimedia.org](http://commons.wikimedia.org/)
        
    - 휘발성이라는 특징을 가지는 RAM의 단점으로 인해 개발되었다.
    - 컴퓨터의 데이터를 반영구적으로 저장할 수 있으며, 데이터 저장소로 사용된다.
    - RAM에 비해 데이터 접근 속도가 매우 느리다. (5 ~ 35ms)
    - 디스크와 헤드가 매우 가깝게 붙어있기 때문에, 충격과 진동에 약하다.
4. Flash Memory
    - HDD와 마찬가지로 반영구적으로 데이터를 저장할 수 있는 장치이다.
    - HDD에 비해 빠르고 내구성이 좋다.
    - 비대칭적인 읽고 쓰기 시간을 가진다. Read(~20us),  Write(~200us)
    - 데이터가 블럭 단위로 구성되어 있으며, 데이터를 작성할 때 해당 블럭을 전부 업데이트하게 된다.
    - Write를 할 때마다 내구성이 닳게 되며, 제한이 존재한다. (각 블럭은 100,000번정도 Write되면 더이상 사용할 수 없다.) → wear leveling과 같은 전략으로 블럭을 고르게 사용하는 전략이 필요하다.
5. Solid State Dist(SSD)
    - Flash Memory에 기반한 데이터 저장 기술이다.
    - Flash Memory에 Flash translation layer(FTL)를 추가하여, 마치 HDD같이 동작하도록 구성되었다.

### **Registers**

> **GPT**
>- **레지스터(register)는 컴퓨터 아키텍처에서 사용되는 데이터를 일시적으로 저장하는 작은 고속 메모리 유닛입니다. 레지스터는 CPU(중앙 처리 장치) 내부에 위치하며, 프로세서의 동작에 직접적으로 참여하여 데이터 및 명령어를 빠르게 처리합니다.**
> 
> 
>- **레지스터는 CPU 내부에서 데이터를 저장하고 연산에 사용함으로써 일시적으로 필요한 데이터를 빠르게 접근할 수 있도록 합니다. 이는 메모리와 달리 레지스터가 CPU의 내부에 위치하여 CPU 클럭 주기에 따라 매우 빠른 속도로 데이터에 접근할 수 있다는 의미입니다.**
> 

Register를 사용하는 이유는 Memory를 참조하는 시간이 프로세스를 처리하는 시간보다 더 오래 걸리기 때문에 이를 최적화하기 위함이다.

Register를 사용하면 메모리 참조를 최소화할 수 있기 때문에 더욱 효율적으로 처리할 수 있게 된다. 

Register 중 특별한 몇가지가 아래와 같이 존재한다.

- Program Counter(PC)
    - 다음에 실행할 Instruction(명령어)의 **Address**를 저장한다.
    - CPU는 실행이 목적이기 때문에 → 실행의 시작점이 필요하다(PC)
- Instruction Register(IR)
    - 최근에 실행한 **Instruction**을 저장한다
- Program Status Word(PSW)
    - CPU의 현재 상태를 Flag의 집합으로 저장한다. (Condition Code, Interrupt Masks 등)
- Memory Addres Register(MAR)
    - 다음에 읽거나 써야할 Address를 저장한다.
- Memory Buffer Register(MBR)
    - MAR에 있는 Address의 Contents를 저장한다.

## **Instruction Cycle**

이제 Instruction Cycle을 이해하기 위한 대부분의 내용을 정리했으니 한번 순서대로 이해해보도록 하자.

참고로 Instruction Cycle을 Pseudo Code로 어느정도 나타내면 아래와 같다.

```c
while(1) {
	Fetch instruction from memory
	Decode instruction
	Execute instruction
		(Get other operands if necessary)
	Store result
}
```

우선 위에서 예시를 든 Opcode를 사용하겠다.

> 0001은 memory에서 데이터를 Load하여 AC에 넣는 동작이고,
0010은 AC의 데이터를 memory에 Store하는 동작이고,
0101은 AC의 값에 memory의 데이터를 Add 하는 동작과 같이 정의될 수 있다.
> 

그리고 메모리는 다음과 같이 저장되어있다고 가정하고, PC Register에 300이 입력되어있다고 가정한다.

![Untitled](/assets/img/posts/2024-03-06/cycle.png)

- Step 1: 300 Address의 Instruction 1940을 가져온다.
- Step 2: 1940은 Opcode 1과 Address 940으로 이루어진 명령어이다. AC Register에 940 Address의 값인 0003을 Load한다. 300 Instruction이 실행되었으므로 PC에는 다음 실행 Instruction인 301이 들어간다.
- Step 3: 301 Address의 Instruction 5941을 가져온다.
- Step 4: 5941은 Opcode 5와 Address 941으로 이루어진 명령어이다. AC Register에 941 Address의 값인 0002를 Add한다. 301 Instruction이 실행되었으므로 PC에는 다음 실행 Instruction인 302가 들어간다.
- Step 5: 302 Address의 Instruction 2941을 가져온다.
- Step 6: 2941은 Opcode 2와 Address 941으로 이루어진 명령어이다. AC Register에 있는 데이터인 0005를 941 Address에 Store한다. PC에는 다음 실행 Instruction인 303이 들어간다.

간단하게 Machine State만 남기고 추상화하게 되면 위와같이 설명이 가능하다.

해당 과정은 C로 구성했을 때

```c
x = 3;
y = 2;
x = x + y;
```

의 코드가 돌아가는 것임을 알 수 있다.

위처럼 Instruction Cycle은 PC에 들어오는 Address에 따라서 명령어를 실행하고, 데이터를 처리하는 과정을 무한정 반복하는 구조로 구성되어 있다. 이러한 구성을 이해하고 있다면 꼭 위 코드가 아니더라도 충분히 어떤 역할을 하는지 이해할 수 있는 기반이 될 것이다.

첫 번째 글을 정리하는데, 너무 모든걸 담으려고 했던 것 같다. 다음 포스트부터는 좀 더 핵심적인, 필요한 내용만 담도록 노력해야겠다.