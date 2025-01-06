# redis-test

redis-test

## Requirement

- K3s
- Helm

## install

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-redis bitnami/redis --version 20.6.0 -f custom.yaml
helm upgrade my-redis bitnami/redis --version 20.6.0 -f custom.yaml
```

## 네트워크 통신 방향

- redis는 pub/sub
- 데이터 전달 관점에선 둘 다 `Push` 방식이다.

### pub

- **Client → Redis**: 지속적 데이터 전송 (Push)

### sub

- 1. **Client → Redis**: Subscriber로 등록하기 위한 1회 통신
- 2. **Redis → Client**: 지속적인 데이터 전송 (Push)
- Subscriber와 Redis가 다른 서버에 있다면 **양방향 네트워크 인가** 필요
- Client가 먼저 request하지만, Pull이 아님에 주의!
- 초기 구독자 등록은 1회만 필요하며 이후 **Client → Redis 인가 해제**도 이론적 가능
  - 단, 리셋, 재부팅 등 실패 상황 시 재등록이 필요할 수 있으므로 사실상 불가

### Redis의 목적과 Push 통신

- 보통 **Redis와 Subscriber는 상호 네트워크 통신이 자유로운 시스템 or 단일 호스트 내에 위치**한다.
- `Redis는 큰 시스템의 일부로서 캐시 메모리나 메시지 브로커 등으로 사용되는게 주 목적`이기 때문
  - e.g. **웹 서버 부하 완화를 위한 고성능 캐시 메모리로서의 Redis**
- 따라서, Redis와 Subscriber 간 양방향 통신이 요구되는 것은 Redis의 철학에 부합한다.
- 만약 양방향 통신이 불가능한 환경이라면 애초에 Redis보다 다른 툴을 쓰는 것이 적합하다고 판단 가능
- 게다가 Redis는 인메모리방식이라 단독 운용되는 빅데이터급 도구라고 보기는 힘듦. Kafka 등 대부분 빅데이터 플랫폼은 디스크 기반으로 설계됨
  - Redis도 디스크 사용과 클러스터링을 통한 증설이 가능하나 나중에 추가된 기능이며, 이런 것들이 Redis의 주 목적이라 보기 힘듦
  - `Kafka, RabbitMQ와 비슷한 역할을 수행하면서 경량의 플랫폼이 필요할 때` 사용하기 딱 좋음!

## UI

- 공식지원: RedisInsight
- 서드파티: Medis, Redis Desktop Manager (RDM)

## redis-cli

- redis 클라이언트 도구
- 인터프리터 방식
- cli는 개발용, 단순 테스트용에 가까움
  - 운영시 UI 사용
  - 실사용시 클라이언트 앱 코드에서 라이브러리로 접근 (redis-py 등)
- redis 설치시 포함(개별 설치가 불가능한건 아닌데 일반적이지 않음)
- 서드파티 cli가 있지만 특수목적이 아닌 이상 그냥 redis-cli를 많이 씀

### 기본 테스트 (Key-Value 저장, 조회, 삭제)

- 대화형 모드

```sh
# Interactive Mode
# redis 접속
redis-cli
```

```redis-cli
SET mykey "Hello, Redis!"
GET mykey

KEYS *
EXISTS mykey

DEL mykey
GET mykey
```

- 비대화형 모드

```sh
# Non-interactive Mode
# redis-cli 진입시 주석이 불가능하므로 이게 나을 수 있음
redis-cli SET mykey "Hello, Redis!" && redis-cli GET mykey

# 따옴표 필요한 경우
redis-cli KEYS '*'
```

```sh
redis-cli <<EOF
SET mykey "Hello, Redis!"
GET mykey
DEL mykey
EOF
```

- 키 타입 확인 예

```redis-cli
TYPE redis-test.1735896660.0
```

- 전체 key 목록 조회(해시 타입)

```redis-cli
HGETALL "redis-test.1735896660.0"
```

- key에서 특정 필드 값 가져오기(해시 타입)

```redis-cli
HGET "redis-test.1735896660.0" "field_name"
```

## redis의 데이터 타입

- key 마다 타입 지정이 되어있다.
- Kafka와 같은 실시간 스트림 데이터를 redis로 받으려면 Hash, List ,Stream 등 타입이 적절
  - 단순 String의 경우 key에 신규값을 쓰면 덮어씌워진다.
  - 타 플랫폼의 계층구조를 다음과 같은 key 이름으로 받는 것이 일반적이며, 필요에 따라 key 이름과 타입을 다르게 설계 가능
  - e.g. `kafka:topic1:partition0`
  
| **데이터 타입**     | **설명**                         | **사용 사례**             |
|---------------------|----------------------------------|---------------------------|
| String              | 단일 값 저장                    | 캐시, JSON, 숫자 연산     |
| Hash                | 필드-값 쌍 저장                 | 사용자 정보, 설정 저장     |
| List                | 순서 있는 리스트                | 작업 큐, 대기열           |
| Set                 | 중복 없는 집합                  | 유니크 태그, 아이디 저장   |
| Sorted Set (ZSet)   | 정렬된 집합                     | 리더보드, 점수 기반 정렬  |
| Stream              | 로그/메시지 스트림              | 실시간 데이터 처리        |
| Bitmap              | 비트 수준 데이터 저장           | 출석 체크, 상태 추적      |
| HyperLogLog         | 근사 중복 요소 계산             | 고유 방문자 수 계산       |
| Geospatial          | 위치 데이터 및 거리 계산        | 지도 서비스, 근처 검색    |
| Pub/Sub             | 메시지 발행 및 구독             | 알림 시스템, 실시간 채팅  |
