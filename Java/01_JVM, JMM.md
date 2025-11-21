## JVM (Java Virtual Machine) 이란?

- Java 프로그램이 운영체제에 의존하지 않고 어디서든 돌아갈 수 있게 하는 자바 가상 머신
- 운영체제에 독립적이라는 특징을 가지고 있다.
- JVM이 바이트코드(.class)를 런타임에 해석하거나 JIT 컴파일해서 OS에 맞는 기계어로 실행함.

<img width="492" height="102" alt="Image" src="https://github.com/user-attachments/assets/7fc0a9d8-76dd-451c-b1ca-061f7d663caf" />

## JVM의 구성

<img width="917" height="590" alt="Image" src="https://github.com/user-attachments/assets/5192541d-5663-400d-98f6-dd2e6ec93f05" />

### 자바 컴파일 단계

- 자바 컴파일러( javac) 가 JVM이 이해할 수 있는 바이트코드로 변환해줌.
- **바이트코드(.class)란?**
    - JVM 전용 언어,  JVM만 있으면 어떤 OS에서도 실행 가능하게 만들어주는 기반.

### JVM 내부 구조

- **클래스 로더(Class Loader)**
    - 바이트코드를 JVM 메모리에 올리고 실행 준비를 하는 파트.
        - **Loading**: 바이트코드를 읽어서 메모리에 올림.
        - **Linking**: 의존성,구조 확인/해석
        - **Initialization**: static 변수 초기화등 초기화 진행함.
        
- **실행 엔진(Execution Engine)**
    - **Interpreter (인터프리터):** 바이트코드를 한줄씩 해석하면서 실행한다.
    - **JIT Compiler (Just-In-Time):** 자주 실행되는 코드(핫스팟)를 기계어로 실시간 컴파일
        - (캐시)
        - 인터프리터보다 빠름.
- **[Garbage Collector](https://www.notion.so/Garbage-Collector-2b1b0ab4fb9f804d84f9c5e00ad248f5?pvs=21) (GC)**
    - 필요 없어진 메모리 자동 회수
    - 개발자가 직접 해제 코드 작성 필요 X (C는 free()로 메모리 해제 해줘야함. )

[Garbage Collector](https://www.notion.so/Garbage-Collector-2b1b0ab4fb9f804d84f9c5e00ad248f5?pvs=21)

- **Runtime Data Area (실행 중 사용하는 메모리 영역)**
    - JVM이 실행 중 데이터를 저장하는 실제 메모리 공간
        - Heap, Stack, Method Area 등으로 구성
        - 객체, 클래스 정보 등이 저장됨.

### 전체 흐름 정리!

1. 개발자가 .java 파일 작성
2. javac가 바이트코드 생성 **(컴파일)**
3. JVM이 바이트코드를 **로딩/검증**
4. 인터프리터 +  JIT 컴파일러가 **실행**.
5. GC가 메모리 자동 관리해줌. (필요 없는 메모리 회수, 정리)
6. (필요하면 네이티브 코드 호출)

## JDK / JRE / JVM

<img width="565" height="342" alt="Image" src="https://github.com/user-attachments/assets/e42b5006-1c14-4371-9efc-af177274d991" />

- JDK: 자바 애플리케이션 개발을 위해 필요한 도구들의 집합.
- JRE: 자바 애플리케이션 실행하기 위한 최소한의 환경.
- JVM: 자바 바이트 코드를 실행하는 가상 머신

---

# JMM (Java Memory Model)

### JMM은 무엇이고, 왜 사용해야 하나?

JMM은 멀티스레드 환경에서 스레드의 값이 언제, 어떤 순서로 보이는지에 대한 문제를 정의한 세트. 

- **기본지식**
    - **프로세스**
        - 실행 중인 프로그램(1개)
        - 독립된 메모리 공간 가짐
    - **스레드**
        - **프로세스 내부에서 실행되는** 작업 흐름(실행 단위)
        - **메모리를 공유**
        - 서로 같은 리소스 봄 → 빠르지만 **동기화 문제 발생 가능! →그래서 JMM이 필요하다.**
        

<img width="510" height="560" alt="Image" src="https://github.com/user-attachments/assets/190895d8-0584-4825-9024-fd149638d947" />

다음처럼, main 스레드와 work 스레드는 **메인 메모리를 공유**한다.(메모리 공유)

하지만 각각 자기 캐시 메모리를 가지고 있다. 그래서 다음의 문제 발생 가능.

- main 스레드가 runFlag = false 로 바꿨는데 → work 스레드는 예전 캐시 값 true를 계속 보고 있을 수 있다.
- 회원이 서버에 메시지를 보냈고 그 메시지는 서버에 잘 저장돼 있지만,
매니저 클라이언트가 웹소켓이나 폴링 같은 동기화 로직을 돌리지 않으면
그 메시지를 영원히 못 보게 되는 것과 비슷하다.

→ 이게 JMM이 해결하려는 문제!

### happens-before 이란?

- JMM에서 스레드 간 **작업 순서를 정의**하는 개념

 A 작업이 B작업보다 happens-before 관계에 있다면,
 A작업에서의 모든 메모리 변경 사항은 B 작업에서 볼 수 있다.
 즉, A 작업에서 변경된 내용은 B 작업이 시작되기 전에 모두 메모리에 반영된다.


- 자바의 여러가지 요소를 통해 happens-before 실현할 수 있다.
    - volatile 키워드
        - ex) volatile boolean running = true;
    - synchronized 블록
    - final 필드의 초기화 규칙
        - final 필드는 선언과 동시에 초기화 되기 때문에 안전하다.(값이 변하지 않는다.)

