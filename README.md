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

### 사용 철학 및 환경

- 보통 `Redis와 Subscriber는 상호 네트워크 통신이 자유로운 시스템 or 단일 호스트 내에 위치`한다.
- Redis는 캐시 메모리나 메시지 브로커 등 역할이 메인 목적이기 떄문이다.
  - e.g. **웹 서버 부하 완화를 위한 고성능 캐시 메모리**
- 따라서, Push 방향 데이터 전송은 Redis의 철학에 부합한다.
- 만약 이러한 환경이 아니라면 애초에 Redis보다 다른 툴을 쓰는 것이 적합하다고 판단해볼 수 있음

### 구독자 등록 인가

- 초기 구독자 등록은 1회만 필요하며 이후 **Client → Redis 인가 해제** 이론적 가능
  - 단, 리셋, 재부팅 등 실패 상황 시 재작업 불가로 실사용은 부적합
  