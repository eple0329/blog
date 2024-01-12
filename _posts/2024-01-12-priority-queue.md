---
title: "[자료구조] Priority Queue"
date: 2024-01-11 13:40:00 +0900
categories: [Algorithm, Theory]
tags: [algorithm, theory, datastructure, heap, queue, priority_queue]     
# TAG names should always be lowercase
toc: true
---
# **Priority Queue**
## **1. 정의**
우선순위가 높은(Key 값이 크거나 작은) 데이터가 먼저 Out되는 자료구조이다.

> **GPT**
>- 우선순위 큐는 각 요소에 **우선순위를 부여하여 관리**하며, 높은 우선순위의 요소부터 처리합니다. 작업 스케줄링이나 네트워크 관리 등 다양한 분야에 이용되며, **배열이나 연결 리스트, 힙 등의 자료구조를 이용해 구현**됩니다.

## **2. ADT (Abstract Data Type)**
**최소값을 우선 출력하는 방식 기준**
- (Key, Value)의 Entry로 값을 저장한다.
- insert(e): Entry e를 삽입한다.
- removeMin(): Key값이 가장 작은 Entry를 반환하고, 삭제한다.
- min(): Key 값이 가장 작은 Entry를 반환한다.
- size(): Queue의 크기를 반환한다.
- isEmpty(): Queue가 비었는지 여부를 반환한다.

## **3.1 Unsorted List Based Priority Queue**
- Array, List와 같은 Sequence를 Queue로 사용한다.
- 선언된 Queue에 우선순위에 상관없이 값을 입력한다. → O(1)
- removeMin()이 호출될 때마다 Queue를 돌면서 우선순위가 가장 높은 값을 출력하고 삭제한다. → O(n)
### **Selection Sort**
- 정렬되지 않은 Sequence에서 구현하는 방법이다.

> **GPT** 
>- 선택 정렬(Selection Sort)은 정렬되지 않은 리스트에서 가장 작은 값을 선택하여 정렬된 부분과 교환하는 정렬 알고리즘입니다.
>1. 주어진 리스트에서 가장 작은 값을 찾습니다.
>2. 가장 작은 값과 리스트의 첫 번째 요소를 교환합니다.
>3. 정렬된 부분 리스트를 제외한 나머지 리스트에서 가장 작은 값을 찾습니다.
>4. 가장 작은 값과 정렬된 부분 리스트의 다음 요소를 교환합니다.
>5. 위의 과정을 반복하여 정렬이 완료될 때까지 수행합니다.
>- 이러한 과정을 통해 선택 정렬은 가장 작은 값을 찾아 정렬된 부분의 앞쪽으로 이동시키는 것을 반복하여, 전체 리스트가 정렬될 때까지 진행합니다. 선택 정렬은 비교 횟수가 많아 큰 규모의 리스트에는 비효율적일 수 있지만, 구현이 간단하고 기존의 데이터를 최소한으로 옮기는 특징이 있습니다.

- 아래는 Selection Sort를 적용한 Priority Queue이다.

``` c++
#include <iostream>
#include <vector>

using namespace std;

class Entry{
public:
    int key;
    int value;
    Entry* prev;
    Entry* next;

    Entry(int key, int value){
        this->key = key;
        this->value = value;
        prev = NULL;
        next = NULL;
    }
};

class PriorityQueue{
public:
    Entry* head;
    int entry_size;

    PriorityQueue(){
        head = NULL;
        entry_size = 0;
    }

    void insert(int key, int value){
        Entry* newEntry = new Entry(key, value);
        if(isEmpty()){
            head = newEntry;
        }
        else{
            head->prev = newEntry;
            newEntry->next = head;
            head = newEntry;
        }
        entry_size += 1;
    }

    int removeMin(){
        Entry* tmp = head;
        Entry* minEntry = head;
        while(tmp != NULL){
            if(tmp->key < minEntry->key){
                minEntry = tmp;
            }
            tmp = tmp->next;
        }
        int minValue = minEntry->value;

        Entry* prev = minEntry->prev;
        Entry* next = minEntry->next;
        if(prev != NULL) {
            prev->next = next;
        }
        if(next != NULL) {
            next->prev = prev;
        }
        delete minEntry;
        entry_size -= 1;
        return minValue;
    }

    int min(){
        if(isEmpty()){
            return 0;
        }
        else{
            Entry* tmp = head;
            Entry* minEntry = head;
            while(tmp != NULL){
                if(tmp->key < minEntry->key){
                    minEntry = tmp;
                }
                tmp = tmp->next;
            }
            return minEntry->value;
        }
    }

    int size(){
        return entry_size;
    }

    int isEmpty(){
        return entry_size == 0;
    }

};
```

## **3.2 Sorted List Based Priority Queue**

## **3.3 Heap Based Priority Queue**
