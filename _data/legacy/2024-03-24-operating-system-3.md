---
title: "[운영체제] 3. Cache의  기본 원리에 대해 알아보자"
date: 2024-03-24 22:00:00 +0900
categories: [Study, 운영체제]
tags: [study, operating system, cache]     
# TAG names should always be lowercase
toc: true
---

## **Cache**

### **목적**

앞서 공부했던 [Instruction Cycle](https://nesquitto.github.io/posts/operating-system-1)에서 CPU가 Instruction을 실행하기 위해 Fetch Instruction, Execute Instruction 과정을 거쳐저 진행한다고 했었다.

여기에서 Fetch Instruction을 진행할 때 Cache가 없고 Instruction 하나당 바닐라 시스템이라고 우선 가정을 하면 반드시 최소한 한 번은 메모리에 접근을 해야한다. 따라서 메모리에 접근을 하기 위해서 메모리 버스를 타야하고, Instruction의 종류에 따라서 또 메모리에 여러번 접근을 하게 될 수도 있다.

그렇기 때문에 시스템의 성능을 향상시키기 위해서는 CPU의 성능이 올라가는 것도 중요하지만, 메모리에 접근해서 데이터를 가져오는 속도도 빨라져야 한다. 이러한 속도를 높이기 위해서 Cache를 사용하게 된다.

### **메모리 성능 Factor**

컴퓨터 메모리의 성능을 결정하는 요인은 크게 3가지로 나누어진다.

1. Capacity : 용량은 많을 수록 좋음
2. Cost(per bit) : 비용은 적을수록 좋음
3. Access time : 빠르면 빠를수록 좋음
    - 메모리가 프로세서의 속도를 따라갈수록 좋은 성능을 기대할 수 있다.

### **현대 메모리 기술**

- SRAM : 면적당 크기가 작고 빠른 접근이 가능하지만, bit당 가격이 비싸다.
- DRAM : 면적당 크기가 크고, 상대적으로 SRAM에 비해 느리지만, bit당 가격이 싸다.

이렇게 두 종류의 RAM을 사용하는데, 이 중 하나의 싱글 메모리로 구성하기에는 가격이 엄청 비싸지거나, 성능이 느리거나 하기 때문에 하이브리드 방식으로 SRAM, DRAM을 함께 사용해서 구성한다.

그리고 이를 효율적으로 사용하기 위해 좀 더 빠른 SRAM을 좀 더 CPU 근처에 두고, DRAM이 SRAM을 보완해주는 방식으로 발전하며, Cache라 함은 이러한 SRAM과 DRAM을 이용하는 방식이다.

![Untitled](/assets/img/posts/2024-03-24/1.png)

## **Memory Hierarchy**

위처럼 CPU와의 거리가 Cache가 가장 가깝고, Disk가 가장 멀리 존재하는데, CPU에 가까이 존재할수록 상위 계층 메모리, 멀리 존재할수록 하위 계층 메모리라고 한다. 이렇게 구성을 하고 적절히 배치를 잘 하게 된다면, 프로세스 처리 상에서 상위 계층의 Access Time을 가지면서 하위 계층의 Capacity를 가지는 효과를 볼 수 있다.

이제부터 어떻게 구성을 해야 이런 효과를 가질 수 있는지에 대해서 설명을 진행할 것이다.

위에서 언급한 효과를 보기 위해서는 최대한 하위 계층의 메모리에 접근하는 것을 최소화해야한다. 즉, 어떤 데이터가 액세스 되었을 때 항상 상위 계층의 메모리에 있도록 하는 전략이 필요하다.

이러한 문제를 직관적으로 바라보면 단순하게 자주 사용하는 데이터를 상위 계층 메모리에 올려두면 된다. 이러한 것을 바로 Locality이다. 

- Locality of reference(참조의 지역성): 프로그램은 특정한 시간에 상대적으로 작은 영역의 주소 범위에만 접근하는 경향이 있다.

Locality는 크게 시간과 공간, 두가지로 나눌 수 있다.

- Temporal Locality(시간 지역성): 한번 Block이 참조될 경우, 근시간에 다시 참조되는 경향이 있다.
    - ex. 반복 루프(for, while)
- Spatial Locality(공간 지역성): Block이 참조되었을 때, 주소 근처의 Block이 참조될 확률이 높다.
    - ex. Array, Table

## **Average Access Time**

이러한 참조와 관련하여 분석을 하기 위해 Hit과, Miss라는 용어가 존재한다.

- Hit: Access된 데이터가 상위 계층 메모리에 있을 경우를 의미한다.
    - H(Hit rate): 상위 레벨에서 발견된 모든 메모리 엑세스의 비율
    - T1: 상위 계층 메모리 접근 시간
    - T2: 하위 계층 메모리 접근 시간
- Miss: Access된 데이터가 하위 계층 메모리에 있을 경우를 의미한다.
    - M(Miss rate): 1 - H(Hit rate)
- Average Access Time 계산
    - T = H * T1 + (1-H) * T2
    - Level 1의 Access Time → 0.1us
    - Level 2의 Access Time → 1us
    - 95%가 상위 메모리에서 Hit됨 (H = 0.95)
    
    → T = 0.95 * 0.1us + 0.05 * (0.1us + 1us) = 0.15us (상위 계층의 접근 속도와 비슷하다)
    
    ** Miss일 때에도 상위 계층 메모리에 한번 접근한 뒤 하위 계층 메모리로 이동하기 때문에 두 접근 시간을 합쳐야 한다.
    

이러한 접근 시간을 줄이기 위해서는 Hit Rate를 높이는 것보다, Miss Rate를 줄이는 것에 집중해야한다. Hit Rate로 95% → 97%의 상승은 겨우 2%의 상승이지만, Miss Rate로는 5% → 3%로 거의 2배의 속도 향상이 이루어지는 것이기 때문이다.

![Untitled](/assets/img/posts/2024-03-24/2.png)

- T = H * T1 + (1 - H)(T1 + T2)
- T = -T2 * H + (T1 + T2)

Average Access Time은 Hit rate에 따라서 다음과 같은 그래프가 나올 수 있는데, 위의 그래프를 보게 되면, Hit rate가 아무리 안좋아도 T1 + T2의 시간은 보장되며, 가장 좋은 때에도 T1에 **근사한** 시간은 걸린다는 것을 알 수 있다.

## **Cache Memory**

이제 물리적으로 어떻게  상위 계층의 Access Time을 가지면서 하위 계층의 Capacity를 가질 수 있는지 알아보자.

### **Motivation**

프로세서는 Instruction을 Patch하기 위해서 메모리에 한번 접근해야하고, 더 추가적인 접근이 필요할 수 있다. 하지만 프로세서에 비해 메모리에 접근하는 속도는 매우 느리고, 이로써 메모리에 접근하는 속도가 중요한 요소로 작용하게 된다. 이런 접근 속도를 보완할 수 있는 방법이 Cache라고 하는 SRAM과 DRAM이 결함된 요소이다.

### **Solution**

위에서 설명한 Locality를 사용하여 더 작고 빠른 메모리(Cache)로 데이터를 제공해야한다. 자주 Access되는 메모리 Block을 Cache에 넣는 방법을 구현해야하며, 이 방법에는 Temporal Locality를 구현하는 방법과, Spatial Locality를 구현하는 방법이 존재한다.

1. Spatial Locality
- 한 번 데이터에 접근하면, 근처 데이터에 또 접근할 가능성이 높다고 분석하여 접근된 데이터뿐만 아니라 주변 큰 덩어리(Block)을 미리 Cache에 prefetching(선반입) 하는 방법이다.
- Block의 사이즈의 크기는 Cache 데이터를 CPU에서 받는 Register의 크기로 정의하고, 이를 Word라고 한다.
1. Temporal Locality
- 최근에 접근된 데이터는 한 번 더 참조될 확률이 높다고 분석하여, 최근에 사용된 instruction과 데이터를  Cache에 저장하는 방법이다.

### **Cache Design**

Main Memory는 $2^n$개의 Addressable Word로 구성되어 있다. (Size = $2^n$) 그리고 Block의 크기가 K라고 한다면 Main Memory는 $2^n/K$ Block으로 구성되게 된다. (M개의 Block 수)

Cache Memory는 Main Memory보다는 적은 개수의 Block으로 이루어지게 된다. (C개의 Block 수)

따라서 Main Memory에 있는 Cache를 Block 단위로 올릴 수 있도록 설계가 된다. 

가장 쉬운 방법은 Main Memory에 있는 모든 데이터를 Cache Memory에 올리는 방법이다. 하지만 이 방법은 현실적으로 Cache Memory의 크기가 현저히 작기 때문에 모두 올릴 수는 없다. 따라서 많이 접근되는 Hot Data들을 Cache Memory에 올리는 전략이 중요하다.

일반적으로 Main Memory에서 Cache Memory로 데이터를 넣는 방식을 Mapping Function이라고 부른다. 데이터를 Cache의 어느 위치에 넣을지 결정한다는 의미이다. 이 방식은 Cache의 Block이 들어가는 위치에 Tag라는 값을 같이 넣게 되는데, 이 Tag가 Main Memory에서 특정한 Block의 데이터가 해당 위치에 들어있다는 것을 식별하는 역할을 한다.

Cache Mapping 방식은 [이 사이트](https://jschan0911.tistory.com/99)를 참고하면 좋을 것 같다. 

이러한 연관을 하게 된다면 크게 3가지 효과를 얻을 수 있다.

1. Associativity -  연관성
- Mapping Function은 Mapping의 연관도가 높아질수록 유연하게 동작하는데, 유연하게 동작할수록 구현을 위한 비용이 더 높아진다.
1. Cache Size - Cache 크기
- Cache의 Size가 커질수록 Hit Rate가 더 높아진다.
- 하지만, 큰 메모리를 빠르게 이용하는 것은 어려워진다.
- 따라서 어느정도 합리적인 크기의 Cache 크기는 성능 향상에 중요한 영향을 미친다.
1. Block Size -  Block 크기
- Block Size가 커질 때 Hit Ratio는 일시적으로 증가하기는 하지만, 프로그램이 변경될 때마다 Memory에 들어있는 데이터가 모두 Miss가 날 것이기 때문에 성능이 떨어지게 된다.

## **Cache Read Operation**

![Untitled](/assets/img/posts/2024-03-24/3.png)

지금까지 설명한 내용의 전반적인 데이터 호출 방식은 위 사진과 같다.

1. 접근해야 하는 주소를 CPU로부터 받는다.
2. 해당 데이터 주소가 Cache의 Block 안에 있는지 확인한다.
3. 만약 존재한다면 해당 부분의 데이터를 가져와 CPU에 보낸다.
4. 그렇지 않다면, Main Memory에 접근한다.
5. 데이터를 가져와서 Cache에 저장할 Slot을 할당하고, Cache에 Block 단위로 저장한다.
6. 동시에 CPU로 데이터를 보낸다.

이처럼 Read Operation은 동작하게 되고, Address Block이 Cache Memory에 존재해서 적중하는 것을 우리는 Hit이라고 부르기로 했다.

## **Cache Write Operation**

Cache는 데이터를 가져오는 것일 때에만 영향을 미치는 것이 아니라, 데이터를 입력할 때(업데이트할 때)에도 영향을 미친다. Write Operation에서 전략은 데이터가 Hit일 때와 Miss일 때로 나뉘어 사용될 것이다.

1. Hit 상황에서의 전략
- Write-through
    - 업데이트 되는 즉시 Cache 뿐만 아니라 Memory에도 반영하는 방식이다.
    - Block이 업데이트 될 때마다 Writing 과정을 거치게 된다.
    - Cache 성능을 기대하기 힘들다.
- Write-back (Delayed Write)
    - Cache만 우선 반영하고, Line이 변경될 때까지 메모리에는 Write 를 지연시킨다.
    - Dirty Bit를 추가하게 된다. (메모리와 캐시의 데이터가 다른지 여부를 저장)
    - 일관성 문제가 발생할 수 있다. (False Sharing 문제)
    - Multi-Processor Operation이나 I/O HW Modules에서 오는 DMA(Direct Memory Access)에 문제가 발생할 수 있음
1. Miss 상황에서의 전략
- Write-allocate
    - Cache에 존재하지 않으면, Memory에 Write하고 Cache에 Load한다.
    - Temporal Locality를 위해서 Cache에 Load하게 된다.
- No-write-allocate
    - Memory에만 Load하고, Cache에는 올리지 않는 방식이다.

위처럼 Write 방식에서의 여러 전략이 존재하는데, 이 중 일반적으로 Write-back과 Write-allocate 방식을 많이 사용한다.

이렇게 여기까지 Cache에 대해서 간략하지만 정리를 해보았다. 이렇게 정리된 나의 지식이 미래에 학습할 기술에 대한 이해도를 높이고, 쉽게 떠올리는데 도움이 되길 바란다.