---
title: "[운영체제] 2. Interrupt의 발생에 대해 알아보자"
date: 2024-03-14 20:00:00 +0900
categories: [Study, 운영체제]
tags: [study, operating system, interrupt]     
# TAG names should always be lowercase
toc: true
---

## **Interrupt란?**

### **정의**

>**GPT**
>- "Interrupt"라는 말은 컴퓨터 과학과 프로그래밍에서 자주 쓰이는 용어입니다. 간단히 말해서, Interrupt는 컴퓨터의 프로세서가 현재 진행 중인 작업을 잠시 중단하고, 급하거나 중요한 작업(예: 사용자의 입력, 하드웨어 신호 등)을 처리한 뒤, 원래의 작업으로 돌아가게 하는 메커니즘을 의미해요. 이를 통해 컴퓨터는 다양한 작업을 효율적으로 동시에 처리할 수 있습니다.
>- 이러한 방식은 컴퓨터가 여러 작업을 더 빠르고 효율적으로 처리할 수 있게 도와주며, 운영 체제나 다른 소프트웨어가 사용자의 입력이나 기타 중요한 이벤트에 빠르게 반응할 수 있도록 합니다.

Interrupt란 간단하게 현재 진행중인 프로세스를 잠시 두고, 다른 급한 일을 먼저 처리한 뒤 다시 진행하는 메커니즘이다.

### **목적**

1. Interrupt의 가장 큰 목적은 Busy waiting을 해소하는 것이다.
- Busy waiting이란 프로그램이 특정한 이벤트나 권한을 얻을 때까지 계속해서 상태를 확인하는 것을 의미한다.
- 다른 작업을 처리하다가 Interrupt가 발생하면 해당 이벤트를 처리하면 되기 때문에 CPU가 다른 작업을 처리할 수 있게 하여 전반적인 효율을 증가시킨다.
- 시스템의 효율성과 반응성을 향상시킨다.
    - Continuously pooling(지속적인 대기) 대신 I/O Device가 CPU에게 필요할 때 신호를 보내는 것을 허용한다.
    - System timer interrupt와 같은 정해진 시간 간격으로 발생하는 이벤트를 신속하게 처리할 수 있게 된다.
- 시스템의 안정성과 보안성을 향상시킬 수 있다.
    - Interrupt를 처리하기 위해 CPU는 kernel mode로 변환하게 되고, kernel mode에서 CPU는 user mode에서는 접근할 수 없는 instruction(명령어)와 메모리에 접근 가능한 권한을 얻을 수 있다.
    - 따라서 일반적인 상황에서 하드웨어 직접제어나 메모리 관리와 같은 영역에 대한 접근을 차단하여 보안을 강화함과 동시에 효율적으로 Interrupt를 처리할 수 있게 한다.

![Untitled](/assets/img/posts/2024-03-14/1.png){: width="400"}

위 그림과 같이 Program을 진행하다가 Interrupt가 발생하면, Interrupt Handler로 이동하여 문제를 처리하고 다시 돌아와 기존의 작업을 계속하게 된다.

1. 다음으로 Timely Servicing, 적시에 서비스를 하기 위함이다.
- 시스템의 전체적인 관점에서 한 프로세스를 계속 수행하기에는 주기적으로 신경써야 할 작업들이 있기 때문에, Interrupt를 통해 주기적으로 개입하여 프로세스 자원을 관리하게 된다.

### **I/O Device vs I/O Controller**

계속 이야기를 진행하기 전에 이 두 용어의 차이에 대해 설명을 하고 넘어가도록 하자.

- I/O Device란 말 그대로 입출력 장치이다. 실제 하드웨어(키보드, 마우스, 프린터 등)로 사용자와 컴퓨터 시스템 간의 데이터를 주고받는 물리적인 장치를 의미한다.
- I/O Controller란 I/O Device와 CPU 사이에서 Interface 역할을 하는 회로나 칩셋을 의미한다. 둘 사이의 데이터 전송을 관리하고 제어하는 역할(에러 체크 등)을 수행한다.

![Untitled](/assets/img/posts/2024-03-14/2.png){: width="400"}

위 그림처럼 I/O 장치와 CPU, Memory는 Bus로 연결되어있다.

우리는 I/O Device를 통해서 입력과 출력을 하지만, 실제 CPU에서는 I/O Controller와 통신을 하면서 프로세스를 처리하게 된다. 하지만 Bus를 통해서 연결이 된 장치들은 CPU 내부에서 프로세스의 처리 속도보다 수백배, 수천배 정도 느리다는 것을 우리는 이전 정리를 통해 알 수 있었다. ([Register 관련 내용](https://nesquitto.github.io/posts/operating-system-1/#registers))

따라서 I/O가 작업을 시작했을 때 CPU 자원을 할당하고, I/O 작업이 끝날 때까지 기다리게 하는것은 자원 효율성 측면에서 바람직하지 않은 방식이며 이를 해결하고자 사용하는 것이 Interrupt이다.

I/O는 필요할 때 CPU에 신호를 보내고, CPU는 계속 처리를 하다가 I/O에서 요청이 왔을 때만 잠깐 처리를 하고 다시 원래 일로 돌아오는 것이다.

### **System Timer란?**

위 글을 읽다보면, System Timer Interrupt란 용어가 나온다. 이는 System Timer 상에서 발생하는 Interrupt라는 뜻으로 System Timer가 무엇인지 알아보도록 하자.

> **GPT**
>- System Timer(시스템 타이머)는 컴퓨터 시스템 내에서 시간을 측정하고 다양한 작업을 예약하기 위해 사용되는 중요한 구성요소입니다. 이 타이머는 컴퓨터의 기본적인 동작과 밀접하게 관련되어 있으며, 운영 체제가 시간을 관리하고 프로세스 간의 스케줄링을 수행하는 데 필수적인 역할을 합니다.

System Timer란 시간에 관련된 역할을 하는 요소이다. 단어의 정의를 통해서 유추할 수 있겠지만 말 그대로 시간에 관련된 것이며, PC 내부에서의 시간을 정밀하게 측정할 수 있게 한다.

커널의 시간은 이 System Timer의 Interrupt를 통해 관리되며 시스템이 시작된 이후 타이머 인터럽트가 발생한 횟수를 저장하는 Jiffies로 측정을 한다. 시스템이 시작된 이후에 측정을 진행하는 것이기 때문에 상대적인 시간을 의미한다.

이 Jiffies를 사용하여 특정 작업을 예약하거나, 시간 차이를 계산하는 등 시간과 관련된 작업에 활용할 수 있다.

반대로 RTC라는 Real Time Clock 하드웨어가 있는데, 이것은 절대 시간을 저장하며 전원이 나가도 배터리를 통해 항상 시간을 유지하게 된다.

### **Process Context, Interrupt Context**

Interrupt가 발생했을 때 이미지를 보면 Process Context에서 Interrupt Context로 넘어가는 것을 볼 수 있다.

이 두가지는 시스템이 코드를 실행하는 환경을 나타내는 용어로 일반적인 상황에서는 Process Context, Interrupt가 발생했을 때에는 Interrupt Context가 실행되게 된다. 따라서 Interrupt Context는 기존의 Process와는 무관하며, 빠르게 처리하고 Process Context로 돌아와야 한다.

> **GPT**
>- 프로세스 컨텍스트 (Process Context)
>- 프로세스 컨텍스트에서는 커널이 사용자 프로세스의 일부로 실행됩니다. 이는 사용자 프로세스가 시스템 호출을 통해 커널 함수를 호출할 때 발생합니다. 이 컨텍스트에서 커널은 현재 실행 중인 프로세스의 문맥(메모리, CPU 상태 등)을 가지고 있으며, 해당 프로세스의 자원과 제한을 준수합니다. 프로세스 컨텍스트에서는 프로세스가 블로킹 연산(예: 파일 시스템 접근, 네트워크 요청 등)을 수행할 수 있으며, 이러한 연산이 완료될 때까지 대기할 수 있습니다.
> 
>- 인터럽트 컨텍스트 (Interrupt Context)
>- 인터럽트 컨텍스트에서는 커널이 직접적인 프로세스의 문맥 없이 실행됩니다. 이는 하드웨어 인터럽트나 소프트웨어 인터럽트가 발생했을 때 일어납니다. 인터럽트 컨텍스트에서는 현재 실행 중인 프로세스를 일시 중지하고, 인터럽트 처리 루틴을 실행합니다. 이 환경에서 커널은 특정 프로세스의 문맥이 아닌, 시스템 전체의 문맥에서 작동합니다. 인터럽트 컨텍스트에서는 블로킹 연산을 수행할 수 없으며, 가능한 한 빨리 인터럽트를 처리하고 원래의 작업으로 복귀해야 합니다.

## **Interrupt Mechanism**

Interrupt Cycle은 다음과 같이 진행된다.

![Untitled](/assets/img/posts/2024-03-14/3.png){: width="400"}

기존 프로세스 설명은 [Instruction Cycle](https://nesquitto.github.io/posts/operating-system-1/#instruction-cycle)에 있으니 참고하면 되며, 해당 설명에 Check Interrupt 과정이 추가된 것이 바로 위의 이미지이다.

일반적인 Process는 아래의 3단계를 따른다.

1. Fetch Next Instruction
2. Excute Instruction
3. Check Interrupt

Interrupt가 발생하지 않았을 때의 상황을 나타내며, 이 경우 명령어를 가져오고, 실행한 뒤, Interrupt 여부를 판단하는 것을 반복한다. 이를 Instruction Cycle이라고 한다.(항상 Interrupt Enabled 상태라고 가정)

Check Interrupt에서 Interrupt가 왔다면 Interrupt를 처리하러 가야하는데, 이 과정에서 바뀌는 것이 바로 **PC(Program Counter) Register**이다. 이 Register를 Interrupt Handler의 주소로 변경을 해줘야 CPU에서 해당 Interrupt를 처리할 수 있을 것이다.

하지만 이 상황에서 Interrupt가 발생했다고 바로 PC를 업데이트해서 이동하면 Loss, 손실이 발생할 것이다. Interrupt Context와 Process Context는 서로 독립적으로 존재하게 되는데, PC를 Interrupt 주소로 업데이트 하게되어 버리면 기존의 Process Context에서 작업하던 위치는 모두 잃게 되고, Interrupt Context에서 원하는 작업을 모두 한 뒤에도 해당 Context에서 계속 작업을 할 것이라는 의미이다.

따라서 PC를 Interrupt Handler 주소로 업데이트 하기전에 기존 Process Context의 PC값을 저장을 해야한다. 이러한 PC를 저장하는 공간을 일반적으로 Control Stack라고 하며, PC값뿐만 아닌 프로그램의 실행 흐름과 관련된 전반적인 정보를 저장하게 된다.

결론적으로 Interrupt가 발생했을 때 처리하는 과정은 다음과 같다.

1. Save PC, PSW to Control Stack
2. Update PC to Interrupt Handler Address
3. Interrupt Handling
4. Control Stack Pop & Update PC

## **PIC(Programmable Interrupt Controller)**

HW에서 Interrupt를 보내는 것은 각각의 하드웨어 장치가 직접 보내는 것이 아니라, PIC라는 별도의 제어기를 통해서 Interrupt를 지원한다. 

PIC는 다른 말로 IRQ Controller라고도 부르는데, IRQ(Interrupt Request)를 제어하는 장치라는 의미이다. 

### **IRQ**

> **GPT**
>- IRQ는 하드웨어나 소프트웨어에서 발생하는 인터럽트 요청을 나타냅니다. 인터럽트는 컴퓨터 시스템에서 다른 작업을 수행하다가 중단되는 것을 의미하며, 예를 들어 하드웨어 장치에서 신호가 발생하거나 소프트웨어에서 특정 이벤트가 발생할 때 인터럽트 요청이 발생합니다. IRQ는 이러한 인터럽트 요청을 처리하기 위해 사용됩니다.

IRQ는 하드웨어가 보내는 Interrupt 요청 자체를 의미하며, 이러한 IRQ의 목록을 관리하는 하드웨어가 PIC이다.

PIC는 Programmable, 프로그램이 가능한 Interrupt Controller이다. Interrupt를 지원하는 장치가 여러개가 존재하고, 각각 CPU에 연결되기에는 비효율적이기 때문에 PIC를 통해서 IRQ를 한번에 관리하는 것이다.

![Untitled](/assets/img/posts/2024-03-14/4.png)

여기 PIC와 하드웨어, CPU의 상호작용을 나타내는 이미지가 있다. 위와 같이 여러개의 장치가 PIC에 각 핀에 맞에 연결이 되어있다. 이 중 특정한 하드웨어에서 Interrupt 신호를 발생시키면, PIC는 CPU에 INTR(Interrupt Request)라는 bit 신호를 보내게 되고, CPU는 Instruction Cycle을 계속 순회하다 해당 bit를 확인하면 IRQ와 함께 Interrupt Process로 이동하게 된다. (처리하겠다는 응답을 ACK로 반환하는 과정도 포함되어있다.)

이렇게 Interrupt를 처리하러 이동을 하게 되면 위에서 언급했던것과 같이

1. Save PC, PSW to Control Stack
2. Update PC to Interrupt Handler Address
3. Interrupt Handling
4. Control Stack Pop & Update PC

다음의 과정을 거치게 된다.

이 과정에서 한가지를 더 추가하게 되면 INTR을 Enabled 상태로 유지할지 Disabled 상태로 변경할지에 대한 Flag를 PSW에 반영하는 단계가 존재하는데, 이는 방식에 따라서 다르게 동작한다.

가장 간단한 방식으로는 Interrupt를 처리할 때에는 INTR을 Disable로 변경하고 처리한 이후에 Enable로 변경하는 방식이 있을 것이다. INTR을 Disabled 하게되면 CPU에서는 Fetch → Execute → Fetch의 과정만을 반복하게 된다. (아래에 나오지만 Non-Maskable Interrupt가 존재하기 때문에 Disabled 상태여도 Interrupt Check의 과정은 항상 거치게 된다.)

그러면 과정은

1. Save PC, PSW to Control Stack
2. Update PC to Interrupt Handler Address & INTR Disable
3. Interrupt Handling
4. Control Stack Pop & Update PC & INTR Enable

다음과 같은 과정으로 Interrupt를 처리하게 된다.

### **기능**

PIC의 기능은 크게 2가지로 존재한다.

1. Translation
- I/O Devices에 IRQ를 제공하여, Interrupt 신호를 Interrupt Vector 형식으로 변환하는 기능
1. Maskable
- IRQ중 Maskable Interrupt를 Masking하기 위한 기능
- 외부 Interrupt는 Maskable Interrupt와 Non-Maskable Interrupt로 구분된다.
- Maskable Interrupt: INTR을 Enable/Disable 할 수 있는 Interrupt
- Non-Maskable Interrupt: INTR이 항상 Enable인 Interrupt (하드웨어 오류 등)

### **Interrupt Handler vs ISR(Interrupt Service Routine)**

지금까지는 Interrupt를 처리하기 위해 Interrupt Handler로 이동한다고 했는데, 정확하게는 Interrupt Handler 내부의 각 장치에 따른 적절한 ISR 함수를 호출한다.

Interrupt Handler는 Interrupt가 발생했을 때 이동하는 말 그대로의 처리를 위한 공통적인 창구이고, 이 Interrupt Handler 내부에는 Interrupt Service Routine이라는 함수가 장치별로 각각 존재한다. 컴퓨터 코드 동작으로 비슷하게 설명하면 코드에 각 장치별로 Interrupt Service Routine 함수가 존재하고 Interrupt Handler가 Switch 문으로 동작해서 장치에 해당하는 함수를 실행하도록 하는 과정이라고 보면 될 것 같다.

Interrupt Handler: Interrupt가 발생했을 때 실행되는 코드 (Common Operation)

ISR: Interrupt Handler에 의해서 실제로 호출되는 코드 (Specific Operation)

ISR $\sub$ Handler

추가로 IDT라고 Interrupt Describable Table이라는 것도 존재하는데, 이는 ISR에 맞는 Function Address가 저장된 Table로 IRQ 각각에 맞는 ISR Function을 매칭할 수 있도록 해준다.

## **Interrupt 처리과정 정리**

1. HW Interrupt 발생
2. CPU Processor에서 Instruction Cycle을 돌고 Check Interrupt 과정으로 진입 
3. Processor에서 Interrupt 발생 확인
4. 현재 Process의 PSW, PC 데이터를 Control Stack에 저장 (HW측면으로)
5. Interrupt Context로 이동하기 위해 PC 값을 해당 주소로 업데이트
6. Interrupt Context로 이동 후 기존 Process의 state information을 모두 저장
7. ISR function을 통해 Interrupt 처리
8. 기존 Process state information 복구
9. 기존 PSW, PC 복구 (Process Context로 이동) (HW측면으로)

## **Asynchronous/Synchronous Interrupt**

지금까지 우리는 HW에서 발생하는 외부 Interrupt의 발생 과정에 대해서 이해해보았다. 하지만 Interrupt는 외부에서만 발생하는 것이 아니라 내부 처리 과정에서도 발생할 수 있는데, 이를 구분하여 다음과 같이 설명한다.

Asynchronous Interrupt - 외부 Interrupt, 일반적으로 Interrupt라고 부른다.

- 비동기적으로 발생하는 Interrupt(Instruction Cycle이 끝나고 Interrupt Check 과정에서 체크 후 처리)
- 하드웨어 디바이스에서 발생한다.

Synchronous Interrupt - 내부 Interrupt, 일반적으로 Exception이라고 부른다.

- 동기적으로 발생하는 Interrupt(발생 즉시 처리)
- Process Context에서 처리 도중 발생한다.
- divide by zero, segmentation fault 등의 예외가 존재한다.
- System Call도 Exception에 포함된다.

## **Interrupt를 사용하는 이유?**

![Untitled](/assets/img/posts/2024-03-14/5.png)

Interrupt를 사용하는 이유는 위에서도 언급했다시피 자원의 Utilization(효율성)을 높이기 위해서이다.

위의 이미지로 처리 시간이 짧은 I/O 처리가 있고, 처리 시간이 긴 I/O 처리가 있다고 하자

1. Short I/O Wait
- Interrupt를 사용하는 경우에 자원을 더 효율적으로 사용하여, Process의 대기시간이 존재하지 않고 병렬적으로 모든 처리를 마무리하였음을 볼 수 있다.
1. Long I/O Wait
- 마찬가지로 Interrupt를 처리하는 경우에 자원을 더 효율적으로 사용하며, I/O의 처리시간이 좀 길긴 하지만 그럼에도 처리 시간에서 확연한 차이가 존재한다는걸 볼 수 있다.

이러한 이유로 Interrupt는 자원의 효율성을 위한다면 필요한 기술인 것을 알 수 있다.

## **여러개의 Interrupt가 들어올 때 처리 방식**

1. Sequential Interrupt Processing
- 순차적으로 들어온 순서대로 처리하는 방식이다.
- 한개의 Interrupt를 처리할 때에는 새로운 Interrupt Request를 무시하고 Interrupt가 끝난 다음에 다음 Interrupt를 처리하는 방식이다.
- 우선순위를 고려하지 않는 방식이다.
1. Nested Interrupt Processing
- Interrupt를 중첩하여 처리할 수 있는 방식이다.
- 한개의 Interrupt를 처리하는 동안 더 높은 순위의 Interrupt가 발생하면 그 새로운 Interrupt의 위치로 또 이동해서 처리하고 돌아와서 나머지 Interrupt를 처리하는 방식이다.
- 우선순위를 고려하여 처리할 수 있다.

## **Interrupt Mechanism의 장단점**

### **장점**

- System Utilization이 대부분의 상황에서 좋아진다.
    - 이런 장점이 단점을 모두 커버한다.

### **단점**

- Control Stack의 공간 비용이 발생한다.
- 불필요한 오버헤드를 발생시킨다.
    - Kernel Mode와 User Mode를 왔다갔다하는 시간비용이 발생한다.

이렇게 Interrupt의 처리 과정부터 세세한 부분까지 모두 살펴보았다. 이렇게 배우면서 점점 내가 원래 가지고 있던 지식에 살이 붙고 있는 느낌이 들었다. 앞으로의 OS 강의가 기대된다.