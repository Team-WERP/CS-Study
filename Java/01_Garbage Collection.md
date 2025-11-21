## Garbage Collection (GC) 이란?
> 자바의 메모리 관리 방법 중 하나

- JVM의 Heap 영역에서 동적으로 할당했던 메모리 중 **필요 없게 된 메모리 객체를 모아 주기적으로 제거하는 프로세스**

- C / C++ 언어에서는 개발자가 직접 메모리 해제 (**`free()`**)
- Java에서는 GC가 메모리 관리를 대행해주기 때문에 한정된 메모리를 효율적으로 사용
- 개발자 입장에서 메모리 관리, **메모리 누수(Memory Leak)** 문제에서 대해 관리하지 않아도 된다.
- Python, JS, Go 언어 등 프로그래밍 언어에서 GC가 기본으로 내장

<br/>

**⛔ 단점** 

- 메모리가 언제 해제되는지 정확하게 알 수 없어 제어하기 힘들다.
- GC가 동작하는 동안에 다른 동작을 멈추기 때문에 오버헤드가 발생 (Stop The World)
    - **STW(Stop The World) 개념**
        - **GC를 수행**하기 위해 **JVM이 프로그램 실행을 멈추는 현상**
        - GC가 작동하는 동안 **GC 관련 Thread를 제외한 모든 Thread는 중단**
        - GC가 완료되면 작업을 재개
        - GC의 성능 개선을 위해선 **STW의 시간을 최소화** 시키는 것

➡️ GC가 자주 실행되면 **성능 하락**의 문제로 이어진다.

<br/>


## GC 대상

> 특정 객체가 Garbage인지 아닌지 판단하는 기준은 **도달성**, **도달 능력**(Reachability)

- 객체에 레퍼런스가 있다면 **`Reachable`** 로 구분되고, 유효한 레퍼런스가 없다면 **`Unreachable`** 로 구분하여 수거한다.
- **`Reachable`** : 객체가 참조되고 있는 상태
- **`Unreachable`**: 객체가 참조되고 있지 않은 상태 (GC의 대상)


- JVM 메모리에서 객체들은 실질적으로 Heap 영역에 생성되고  Method Area이나 Stack Area에서는 Heap Area에 생성된 객체의 주소만 참조하는 형식으로 구성된다.
    - Heap 영역
        > 참조형 데이터 객체의 실제 데이터가 저장되는 공간 
        - Stack 영역에서 실제 데이터가 존재하는 Heap 영역의 참조값을 가지고 있다.
        - **`new`** 키워드로 인스턴스를 생성할 때 Heap 영역에는 생성된 객체가 저장되고 Stack 영역에서 생성된 객체에 대한 주소 값이 저장된다.
    - Stack 영역
        > 기본 자료형, 지역 변수, 매개 변수가 저장되는 메모리
        
- Heap Area의 객체들이 메서드가 끝나는 등의 특정 이벤트로 인해  Heap Area 객체의 메모리 주소를 가지고 있는 참조 변수가 삭제되는 현상이 발생하면, 위 그림과 같이 빨간색 객체와 같이 Heap 영역에서 참조하고 있지 않은 객체들이 발생한다.

    → 이러한 객체들을 주기적으로 GC가 제거한다. 

<br/>

## GC 컬렉션 청소 방식
> **`Mark and Sweep`**
- GC가 Unreachable한 객체를 Mark and Sweep 방식으로 청소

### Mark and Sweep
> GC에서 사용되는 객체를 솎아내는 내부 알고리즘
- GC가 동작하는 기초적인 청소 과정

- GC 대상 객체를 식별(Mark)하고 제거(Sweep)하며 객체가 제거되어 파편화된 메모리 영역을 앞에서부터 채워나가는 작업(Compaction)을 수행한다.

**`Mark`** : Root Space으로부터 그래프 순회를 통해 연결된 객체들을 찾아내 각각 어떤 객체를 참조하고 있는지 찾아서 마킹한다. 

**`Sweep`** : Unreachable 객체들을 Heap에서 제거한다.

**`Compact`** : Sweep 후에 분산된 객체들을 Heap의 시작 주소로 모아 압축한다.

- Minor GC에는 Mark and Sweep을 사용하고, Major GC에는 Mark and Sweep and Compact를 사용한다.

<br/>

## GC 동작 과정

### Heap 메모리 구조
> 동적으로 레퍼런스 데이터가 저장되는 공간으로, GC의 대상이 되는 공간
- Heap 영역은 처음 설계될 때 다음 2가지를 전제(Weak Generational Hypothesis)로 설계되었다.
    - 대부분의 객체는 금방 접근 불가능한 상태(Unreachable)가 된다.
    - 오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.

→ **객체는 대부분 일회성되며, 메모리에 오랫동안 남아있는 경우는 드물다.**

이러한 특성을 이용해 보다 효율적인 메모리 관리를 위해 객체의 생존 기간에 따라 물리적인 Heap 영역을 나눴고, Young과 Old 총 2가지 영역으로 설계하였다.

### Young 영역 (Young Generation)

> 새롭게 생성된 객체가 할당(Allocation)되는 영역

- 대부분의 객체가 금방 Unreachable 상태가 되므로 많은 객체가 Young 영역에 생성되었다가 제거된다.
- Young 영역에 대한 GC를 Minor GC라고 부른다.

- 효율적인 GC를 위해 Young 영역을 3가지 영역으로 나눈다.

- Eden
    - new를 통해 새로 생성된 객체가 할당되는 영
    - 정기적인 GC 수집 후 살아남은 객체들은 Survivor 영역으로 보냄
- Survivor 0 / Survivor 1
    - 최소 1번의 GC 이상 살아남은 객체가 존재하는 영역
    - Survivor 0 또는 Survivor 1 둘 중 하나에는 꼭 비어있어야 한다.

### Old 영역 (Old Generation)

> Young 영역에서 Reachable 상태를 유지하여 살아남은 객체가 복사되는 영역

- Young 영역보다 크게 할당되며, 영역의 크기가 큰 만큼 GC는 적게 발생한다.
    - 크게 할당되는 이유
        
        Young 영역의 수명이 짧은 객체들은 큰 공간을 필요로 하지 않으며 큰 객체들은 Young 영역이 아니라 바로 Old 영역에 할당되기 때문이다.
        
- Old 영역에 대한 GC를 Major GC라고 부른다.

### Minor GC 과정

- Young Generation 영역은 짧게 살아남는 메모리들이 존재하는 공간이다.
- 모든 객체는 처음에 Young Generation에 생성된다.
- Young Generation의 공간은 Old Generation에 비해 상대적으로 작기 때문에 메모리 상의 객체를 찾아 제거하는데 적은 시간이 걸린다.

    → Young Generation 영역에서 발생되는 GC를 Minor GC라 불린다.


1. 처음 생성된 객체는 Young Generation 영역의 일부인 Eden 영역에 할당된다. Eden 영역이 다 채워지면 Minor GC가 일어난다.


2. Mark and Sweep 이후에 살아남은 객체들은 Survivor 영역으로 옮겨지고 Age bit 값이 증가하게 된다.



3. 또다시 Eden 영역에 신규 객체들도 가득 차면 Minor GC가 동작한다.



- 살아남은 객체를 옮길 때 두 개의 Survivor 공간 중 이미 채워져 있는 공간으로 이동

4. 이동하기 전에 Survivor 공간이 가득 차므로 GC가 동작한다.


- Survivor 0에서 살아남은 객체들은 Survivor 1으로 이동한다.

### Major GC 과정
- Old Generation은 길게 살아남는 메모리들이 존재하는 공간이다.
- Old Generation의 객체들은 처음에 Young Generation에 의해 시작되었으나 GC 과정 중에 제거되지 않은 경우 age 임계값이 차게 되어 이동된다.
- Major GC는 객체들이 계속 Promotion되어 Old 영역의 메모리가 부족해지면 발생한다.
    - Promotion
        - 객체의 age가 임계값에 도달하면 Old Generation으로 이동하는 것을 말한다.
- Major GC는 Full GC라고도 부르기도 한다.

1. 특정 age bit를 넘으면 Old Generation 으로 이동한다.



2. Old Generation도 가득 차면 Major GC가 동작한다.

- Old 영역에 있는 모든 객체들을 검사하여 참조되지 않는 객체들을 한번에 삭제하는 Major GC가 실행한다.
- Old 영역은 Young 영역에 비해 상대적으로 큰 공간을 가지므로 이 공간에서 메모리 상의 객체 제거에 많은 시간이 걸린다.
    
    → STW 문제가 발생하게 된다.

## 참고 문헌
- https://mangkyu.tistory.com/118
- https://inpa.tistory.com/entry/JAVA-☕-가비지-컬렉션GC-동작-원리-알고리즘-💯-총정리#garbage_collectiongc_이란
- https://lucas-owner.tistory.com/38#google_vignette