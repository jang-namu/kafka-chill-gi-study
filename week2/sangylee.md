# 서론

모든 기업에서 데이터는 매우 중요하다. 하지만 데이터 그 자체로는 별다른 의미를 갖지 않는다.

- 데이터를 생성된 위치에서 분석할 수 있는 위치로 옮기는 작업이 필수

이 작업을 빠르고 효율적으로 할수록 사업적으로 더 높은 가치를 창출해 낼 수 있다. 

- 데이터 중심 기업에서 데이터 파이프라인을 중요하게 생각하는 이유가 된다.

> [!NOTE]  
> 즉 데이터를 이동하는 방식이 데이터 그 자체만큼 중요해짐

</aside>

# 발행자/구독자 메시지 방식

데이터 중심 프로그램에서는 발행(publish)/구독(subscribe) 패턴이 핵심요소이다.

- 이 방식에서는 (sender/publisher)가 메시지를 직접 (receiver/subscriber)에게 전달하지 않는다.
- 발행자는 메시지를 분류하고 구독자는 특정 종류의 메시지를 가져오도록 구독을 한다.


> [!NOTE]  
> 보통 이 방식에는 위 작업을 효율적으로 하기 위해서 **broker**를 두는 경우가 많다.

# 간단한 예시부터

어딘가에 정보를 보낼 필요가 있다고 가정하자.

- 2개 웹에서 정보를 보내기 위해서 metric 서버에 직접 연결했다.

<img width="546" alt="image" src="https://github.com/user-attachments/assets/7c0e634c-02c9-4ccd-b847-3b0c20dfd6fb" />


하지만 시간이 지날수록 더 많은 웹에서 metric 서버에 정보를 보내야 하는 상황이 발생, 또한 이 정보를 받아야 하는 metric 서버가 다양해지기 시작하면 직접 연결을 통한 방식은 다음과 같이 매우 복잡해질 것

<img width="529" alt="image" src="https://github.com/user-attachments/assets/965ab2e9-5ef4-4f3e-ad07-8a0e258fe8d3" />



웹에서 생성한 metric을 전달 받고, metric 서버가 필요할 때 마다 해당 정보를 전달하는 프로그램을 중간에 둬서 해결했다.

- 이게 pub/sub 메시지 시스템의 기본 아키텍처

<img width="524" alt="image" src="https://github.com/user-attachments/assets/14e7bc06-5a7d-44f5-8dd6-6cadc23dd45b" />


## 개별 큐 시스템

각 웹 서버가 생성한 데이터의 종류가 여러가지이고 각 데이터는 서로 다른 서버에 필요하다면 각 데이터 종류마다 서로 다른 큐를 두는 것이 가능하다.

- 모든 데이터를 한 큐에 담아서 처리하는 것은 병목 현상을 일으킬 수 있다.

<img width="517" alt="image" src="https://github.com/user-attachments/assets/124647b9-b482-41eb-b647-ed4aa96f34ff" />


이런 방식은 2번째 그림처럼 일대일 직접 연결 보다 좋지만 다음 문제점이 있다.

- 각 메시지 큐를 개발자가 따로 관리해야 함
- 데이터의 종류가 늘어날 때 마다 큐를 새로 직접 생성해야함

이런 작업을 관리해줄 **중앙 집중형 시스템**이 필요하다.

# 카프카 입문

**아파치 카프카**는 다음 문제를 해결하기 위해서 만들어졌다.

- 데이터가 늘어날 때마다 메시지 큐 운영 오버헤드가 증가

주로 다음과 같이 카프카를 소개한다.

- 분산 commit log ← DB 관점
    - DB가 커밋 로그를 저장하는 이유는 롤백을 통해 데이터의 영속성과 일관성을 보장하기 위해서
- 분산 스트리밍 플랫폼
    - 카프카는 데이터를 순서대로 저장하고 읽어서 일관성을 보장
    - 카프카는 데이터를 분산 저장해서 실패에 대한 오류 복구와 가용성을 확보

## 메시지와 배치

카프카의 데이터 처리 단위는 **message**

- key라는 metadata를 가지고 있음
- 메시지가 저장된 파티션을 지정할 때 key가 사용

카프카는 메시지를 batch 형태로 저장한다. (효율성을 위해)

- 배치는 메시지의 집합
- 하나의 배치는 같은 topic과 파티션을 가지고 있다.

> [!NOTE]
> **배치 크기와** 지연 시간과 처리량 사이의 관계
> - 배치가 클 수록 단위 시간당 처리할 수 있는 메시지 수가 많아짐
> - 배치가 클 수록 개별 메시지가 전파되는 데 오래걸림
> - 배치가 클 수록 처리 능력을 희생해서(압축) 효율적인 데이터 전송 및 저장
> - 배치가 클 수록 메시지 사이 통신 오버헤드를 줄일 수 있음
</aside>

## 스키마

단순한 바이트 배열을 사람이 읽기 쉽게 변화 시키기 위해서는 스키마가 필요함

- json과 xlm은 가장 단순하지만 호환성과 강타입 처리에 어려움 (??)
- Apache Avro → 원래 하둡을 위해 개발된 직렬화 프레임워크 (선호)

## 토픽과 파티션

카프카는 메시지를 **topic**으로 분류한다. 

- DB에서 table, 혹은 파일시스템에서 폴더와 동치

토픽은 여러 **partition**으로 분할된다.

### 특징

<img width="512" alt="image" src="https://github.com/user-attachments/assets/5f0b39d1-24f0-4cb7-8869-6ccaf1f3fde0" />


- 메시지는 항상 파티션의 맨 뒤에 삽입 되는 형식으로만 저장
- 메시지의 읽는 순서는 항상 앞에서 뒤로
- 한 토픽은 여러 파티션으로 이루어져 있다.
- 한 파티션 내에 메시지 순서는 보장되지만 여러 파티션이 있으면 전체 토픽간 메시지 순서를 보장하지 않음
    - 일반적으로 한 주제에는 여러 개의 파티션이 있기 때문에, 단일 파티션 내에서만 전체 주제에 걸쳐 메시지 정렬이 보장되지 않습니다
- 각 파티션은 서로 다른 서버에 위치할 수 있다.
    - 단일 토픽은 수평적으로 확장 가능하다.
- 각 파티션은 서로 복제가 가능하다

### stream

카프카와 비슷한 시스템에서 데이터를 부르는 말

- 파티션의 수와 관계없이 단일 토픽을 고려한다.
- 단일 data 스트림이 producer에서 consuer로 전달된다.

> [!NOTE]
> 즉 스트림이란 프로듀서에서 컨슈머로 이동하는 단일 데이터 흐름을 나타냅니다. 

- 하둡과 다르게 처리되는 방식이기 때문에 뒷장에서 논의해보자

## 프로듀서와 컨슈머

카프카의 두가지 기본 클라이언트 (시스템 사용자)이다.

### 프로듀서가 하는 일

- **메시지 생성**
    - **새로운 메시지를 생성해서 특정 토픽에 전송**
- **파티션 분배**
    - 메시지를 토픽에 있는 모든 파티션에 균등하게 분배
    - 메시지 키와 파티셔너를 사용해서 특정 파티션에 메시지 보내기 가능
    - 메시지 순서를 유지하는데 유리한 특성

### 컨슈머가 하는 일

- **메시지 읽기**
    - 파티션에 생성된 순서대로 메시지를 읽는다
- **오프셋 관리**
    - 이미 소비한 메시지를 추적하기 위해서 관리하며 중지 후 다시 시작해도 위치를 잃지 않는다.
- **컨슈머 그룹**
    - 여러 컨슈머가 함께 토픽을 소비하는 그룹
    - **컨슈머 그룹 내에서 한 컨슈머는 단일 파티션에 반드시 매핑된다. (파티션 소유권) → 즉 한 파티션에 여러 컨슈머가 들어갈 수 없음**
    
> [!NOTE]
> 한개의 컨슈머가 실패하면 그룹의 나머지 멤버들이 실패한 멤버가 소비한 파티션을 재할당해서 실행

> [!NOTE]
> 그렇다고 한 컨슈머가 반드시 단일 파티션에 매핑되는 것은 아니다한 컨슈머가 여러 파티션에 매핑 가능

<img width="528" alt="image" src="https://github.com/user-attachments/assets/763627da-b25c-4e81-b561-1280d7dc2050" />


## 브로커와 클러스터

### 브로커

카프카에서 단일 서버를 의미한다.

- 메시지 저장 및 전송
    - producer 로부터 메시지를 받고 offset을 할당한 다음 디스크에 저장
    - consuer 요청에 응답해서 파티션에 대한 데이터 제공
- 성능
    - 하드웨어의 성능 특성에 따라 수 천개 파티션과 초당 수백만개 메시지 처리 가능

### 클러스터

여러 브로커로 이루어져 있으며 데이터 스트림을 관리하는 주체

- 컨트롤러가 존재
    - 클러스터 내부에서 한 브로커가 자동으로 선출
    - 파티션 할당, 브로커 장애 감지 등 관리 작업 수행
- 파티션이 존재
    - **한 브로커에 의해 소유된다 (컨트롤러가 배정)**
    - 파티션의 소유권을 가진 브로커를 Leader
    - 파티션의 복제를 가진 브로커를 Follower
    - Leader → 파티션의 리더는 모든 읽기 쓰기 요청을 처리
    - Follower → 팔로워는 리더의 데이터를 복제해서 장애시 대체할 수 있다.
- **복제 및 보존**:
    - **복제(Replication)**: 메시지의 중복성을 제공하여 장애 시 데이터 손실을 방지.
    - **보존(Retention)**: 메시지를 일정 기간 동안 저장하며, 주제별로 보존 설정을 구성 가능.
 
## 여러개의 클러스터
클러스터를 여러개 두는 것이 유용한 이유
* 데이터 유형 분리
* 보안 요구사항에 따른 격리
* 다중 데이터센터 구성 (재해 복구)
<img width="660" alt="image" src="https://github.com/user-attachments/assets/ad337118-24d8-4835-9c25-eece41a8c98f" />

Kafka 클러스터 내의 복제 메커니즘은 단일 클러스터 내에서만 작동하도록 설계되어 있어 여러 클러스터 간 복제에는 적합하지 않다.
* 이를 위해 Kafka 프로젝트는 MirrorMaker라는 도구를 제공.
* MirrorMaker는 기본적으로 Kafka 컨슈머와 프로듀서를 큐로 연결한 것
* 한 Kafka 클러스터에서 메시지를 소비하고 다른 클러스터로 생산

MirrorMaker를 사용하면 두 개의 로컬 클러스터에서 메시지를 집계하여 하나의 통합 클러스터로 모으고, 이를 다시 다른 데이터센터로 복사하는 등의 복잡한 데이터 파이프라인을 구성할 수 있다.

이 간단한 구조의 애플리케이션은 강력한 데이터 파이프라인을 생성하는 데 매우 유용

# 카프카를 써야하는 이유
Apache Kafka는 다양한 장점으로 인해 publish/subscribe 메시징 시스템으로 선호.
주요 특징은 다음과 같다.

## 다중 프로듀서 및 컨슈머 지원

- 여러 프로듀서가 동시에 여러 토픽 또는 동일한 토픽에 데이터를 전송할 수 있다.
- 여러 컨슈머가 서로 간섭 없이 동일한 메시지 스트림을 읽을 수 있다.
- 컨슈머 그룹을 통해 메시지 처리를 분산시킬 수 있다.

## 디스크 기반 보존

- 메시지를 디스크에 저장하여 구성 가능한 보존 규칙에 따라 유지
- 컨슈머가 처리 속도가 느리거나 트래픽이 급증해도 데이터 손실 위험이 없다.

## 확장성

- 단일 브로커에서 시작하여 수백 개의 브로커로 확장 가능.
- 클러스터 온라인 상태에서 확장 가능하며 가용성에 영향을 주지 않는다.

## 고성능

- 대규모 메시지 스트림을 쉽게 처리할 수 있다.
- 프로듀서, 컨슈머, 브로커 모두 확장 가능
- 메시지 생성부터 컨슈머 가용성까지 1초 미만의 지연 시간을 제공

## 플랫폼 기능

- Kafka Connect: 데이터 소스와 Kafka 간의 데이터 이동을 지원
- Kafka Streams: 확장 가능하고 내결함성이 있는 스트림 처리 애플리케이션 개발을 위한 라이브러리를 제공

이러한 특징들로 인해 Kafka는 다양한 규모의 실시간 데이터 스트리밍, 이벤트 기반 아키텍처, 데이터 통합, 분석 등에 적합한 솔루션
