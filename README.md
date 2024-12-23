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

## redis-cli

- redis 클라이언트 도구
- redis 설치시 포함(개별 설치가 불가능한건 아닌데 일반적이지 않음)
- 서드파티 cli가 있지만 특수목적이 아닌 이상 그냥 redis-cli를 많이 씀
- 외부 관리 필요시 UI 사용

## UI

- 공식지원: RedisInsight
- 서드파티: Medis, Redis Desktop Manager (RDM)
