- [Fork Join Pool](#fork-join-pool)
- [Virtual Threads](#virtual-threads)
- [GC 알고리즘](#gc-알고리즘)


<br>


# Fork Join Pool
### 등장배경
- 전통적인 스레드가 멀티코어 CPU를 효율적으로 활용하는데 한계가 있어서 등장
    - why?
        - 전통적인 ThreadPoolExecutor는 멀티코어와 스레드풀이 있는 환경에서도 하나의 공통 작업큐만 사용
            - 모든 스레드가 하나의 큐에서 경쟁적으로 작업을 가져옴
            - 큐에서 큰 작업이 먼저 스케줄링 되면 해당 스레드의 전용작업이 되버려서 완료될때까지 잠기고 다른 스레드도 이를 도울 수 없음
            - 스레드 간 작업 재분배 매커니즘이 없어 어떤건 바쁘고 어떤건 놀고있어도 조정이 불가함
### 장단점
- 장점
    - 워크 스틸링: 각 워커 스레드는 자신의 양뱡향 작업 큐가 있고 작업을 LIFO로 가져오고 큐가 비면 다른 바쁜 스레드의 큐에서 FIFO로 작업을 훔쳐옴
    - 분할 정복: 큰 작업을 처리하는 스레드가 작업을 여러개의 서브 태스크로 나눠서 유휴 스레드에게 나눠줄 수 있음
    - 대용량 데이터 처리, 재귀 분할이 가능한 알고리즘, 독립적인 서브태스크로 분할 가능한 작업, CPU 바운드 작업 등이 적합
- 단점
    - 작은 작업에는 분할/병합의 오버헤드가 더 클 수 있음
    - I/O 작업에 비효율적
        - why?
            - I/O 작업은 블로킹이 되기 때문에 워커스레드 자체가 블로킹 되어버림
            - I/O 작업은 블로킹 되버린 스레드의 작업큐는 다른 스레드에서 작업을 훔쳐갈 수 없음
            - Fork Join Pool은 기본적으로 코어 수에 비례한 스레드를 사용하는데 이는 많은 스레드를 필요로 하는 I/O 작업에 부적합


<br>


# Virtual Threads
### 등장배경
기존 자바의 스레드가 OS 스레드와 1:1 매핑되는 방식의 한계를 해결하기 등장
- 스레드 당 대략 1MB의 스택 메모리 소비
- OS 스레드 간 컨텍스트 스위치 비용이 비쌈
- I/O 작업에서 블로킹 되어버림

### 동작 원리
- OS 스레드가 아닌 JVM이 직접 스레드를 관리
- 캐리어 스레드: 실제 OS 스레드로 가상 스레드를 실행하는 매체
- Continuation: 가상 스레드의 실행 상태를 캡처하고 저장하는 메커니즘. 블로킹 작업이 발생하면 가상 스레드의 상태를 저장하고 carrier 스레드를 해제
- pinning: 가상 스레드가 네이티브 메소드나 synchronized 블록 실행 중이라면 언마운트될 수 없음. 이 상태에서는 일반 OS 스레드처럼 동작
- Mounting/Unmounting: 가상 스레드가 실행을 시작하면 carrier 스레드에 마운트 되고 블로킹 작업에서는 언마운트됩니다.

### flow
1. 가상스레드에서 블로킹 작업 발생
2. 가상 스레드 실행 상태 저장 (Continuation)
3. 캐리어 스레드 해제
4. 블로킹 작업 완료
5. 가상 스레드 재개

### 장단점
- 장점
    - 동시 연결 처리 탁월
        - 스프링 MVC에서 요청마다 가상 스레드가 생성 됨
    - 메모리 효율성 (약 2KB)
    - 기존 블로킹 I/O 코드와 같이 사용 가능
    - 스택 트레이스가 완전히 보존되어 디버깅 용이
- 단점
    - I/O 바운드에는 탁월하지만 CPU 바운드 에서는 여전히 코어 개수가 중요
    - synchronized는 핀닝을 발생시켜 언마운트 안됨 (일반 OS 스레드처럼 블로킹)
    - 많은 가상스레드에서 스레드로컬을 사용하면 메모리 부담
    - 아직 프레임워크/라이브러리에서 최적화 되지 않음

### why
비슷한 해결책을 제시하는 리액티브 프로그래밍(web flux)이 있는데 냅두고 왜 나왔지?
- 언어 수준에서 지원하기 위해?
- 러닝커브가 낮고 기존 MVC 코드와의 통합이 용이해서?
    - I/O 비동기 처리 자체를 추상화?
- 스택트레이스가 온전히 존재해 디버깅 하기가 쉬워서?


<br>


# GC 알고리즘
### 동시성의 차이
- 병렬 처리 (Paraller): 여러 GC 스레드가 동시에 GC를 수행하지만 이 동안 Stop-the-world
- 동시 처리 (Concurrent): GC작업이 어플리케이션 스레드와 함께 수행됨. 즉 어플리케이션이 실행되는 동안 GC가 백그라운드에서 실행됨

### 동시성의 trade-off
- 동시성이 높을수록 stop-the-world는 줄어들지만 다른 비용 발생
    - CPU 오버헤드: 동시작업은 GC가 어플리케이션과 CPU 리소스를 공유해야 함
    - 처리량 감소: 동시성이 높을수록 처리량은 감소

### Parallel GC
- 최대 처리량을 목표로 설계
- 병렬로 GC작업을 수행하기 때문에 동시성이 없음
- GC 작업과 어플리케이션의 실행이 동시에 일어나지 않음

### G1 GC
- 동시성과 처리량의 조화를 목표로 설계
- 처음으로 동시성을 도입한 CMS(현재 지원종료) 이후 향상된 부분적인 동시성 제공
- 하지만 일부 중요한 단계에서는 Stop-the-world가 필요함

### ZGC
- 동시성 극대화를 목표로 설계
- 동시성을 극대화함. 거의 모든 GC작업이 어플리케이션 스레드와 동시에 실행
- 마이크로초 단위의 매우 짧은 Stop-the-world가 필요함


<br>