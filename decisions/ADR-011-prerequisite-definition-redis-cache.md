# ADR-011: 선행지식 설명은 ElastiCache for Valkey에 Document·용어 단위로 30일 캐시

## 1. Overview

- Date: 2026-09-21
- Status: Accepted
- Deciders: 근흐흐
- Tracking: [FT-012 선행지식](../features/FT-012-선행지식.md)
- Related: [AWS Infrastructure](../architecture/aws-infrastructure.md), [BE ↔ AI OpenAPI](../contracts/backend-ai/openapi.yml)

## 2. Context

선행지식 하이라이트는 knowledge compile 단계에서 미리 만들어지지만, 영문·국문 설명은 사용자가 하이라이트를 처음 누를 때 AI가 생성한다. 설명 생성에는 지연과 모델 비용이 들며 같은 원본 Document를 읽는 여러 사용자가 같은 용어를 반복해서 요청할 수 있다.

설명은 영구 원본 데이터가 아니라 다시 생성할 수 있는 파생 데이터다. 논문에 등장하는 용어의 설명이나 예시는 시간이 지나며 달라질 수 있으므로 일정 기간 뒤에는 새로 생성할 수 있어야 한다. 동시에 기능별 파생 데이터를 PostgreSQL에 계속 추가하지 않고 캐싱 책임을 별도 계층으로 분리하려 한다.

## 3. Decision

선행지식 설명은 PostgreSQL에 영구 저장하지 않고 Redis-compatible cache에 저장한다. 배포 환경은 Amazon ElastiCache for Valkey의 node-based cluster를 사용하고, local은 Valkey-compatible container를 사용한다.

- DEV·PROD는 각각 `cache.t4g.micro` 1대로 시작한다. cluster mode는 끄고 TLS를 켜며, 파생 캐시라 백업은 두지 않는다.
- 캐시 범위는 사용자별 Paper가 아니라 모든 사용자가 공유하는 원본 Document다.
- 같은 Document 안에서 동일한(정규화 결과가 같은) 선행지식 문구는 등장 위치와 사용자에 관계없이 하나의 설명을 공유한다.
- 문구는 Unicode NFC 적용, 앞뒤 공백 제거, 연속 공백 축소와 대소문자 통일을 거쳐 정규화한다. 어간 추출이나 의미 기반 병합은 하지 않는다.
- 캐시 키는 `prerequisite-definition:v1:{documentId}:{generatorVersion}:{normalizedTermHash}`다. `v1`은 캐시 value schema version이고 generator version은 설명 생성 프롬프트·모델·출력 규칙의 버전이다.
- knowledge compile revision은 키에 넣지 않는다. Document가 동일한 파일 바이트의 공유 단위이고, 설명 생성 규칙의 변경은 generator version으로 무효화한다.
- 배포 설정의 동일한 `PREREQUISITE_KNOWLEDGE_AGENT_ACTIVE_VARIANT`를 Backend와 AI에 주입한다. Backend는 캐시 키에, AI는 프롬프트·모델 조합 선택에 사용한다. 기존 BE↔AI 선행지식 계약에는 generator version 필드를 추가하지 않는다.
- value는 `definitionEn`, `definitionKo`, `generatedAt`만 포함하는 JSON이다. 팝오버 제목은 캐시가 아니라 현재 하이라이트의 원문 `text`를 그대로 사용한다.
- 성공한 설명만 저장한다. AI 실패 결과는 저장하지 않는다.
- 기본 TTL은 생성 시점부터 30일로 설정하며 조회로 만료 시각을 연장하지 않는다. TTL은 배포 설정값으로 주입한다.
- 만료되거나 축출된 값은 cache MISS로 처리해 다시 생성한다.
- Backend는 사용자당 선행지식 설명 생성을 동시에 하나만 허용한다. 캐시 HIT는 이 상한을 소모하지 않는다.
- 서로 다른 사용자의 동일 identity 동시 MISS를 하나의 AI 호출로 병합하지 않는다. MVP는 이 중복 생성을 허용하고 먼저 저장된 성공 결과를 이후 요청이 재사용한다.
- 캐시를 읽기 전에 사용자의 Paper 접근 권한을 검증하고 공유 Document를 해석한다. 캐시 identity에는 사용자 식별자를 넣지 않는다.
- 선행지식 설명 생성은 사용자 AI 질의 사용량을 차감하지 않는다. 실제 cache MISS에서 발생한 추정 비용은 운영 관측용으로 기록한다.

## 4. Options Considered

### ElastiCache for Valkey node-based cache — 채택

- TTL과 eviction이 설명의 재생성 가능성, 30일 갱신 정책과 맞는다.
- 공유 Document의 같은 용어를 여러 Backend 인스턴스와 사용자 사이에서 재사용할 수 있다.
- 캐싱 책임을 PostgreSQL의 영구 도메인 데이터와 분리한다.
- Valkey는 Redis-compatible protocol을 사용하면서 AWS의 Redis OSS engine보다 낮은 node 요금을 제공한다.
- node-based와 cluster mode disabled 구성은 일반적인 Redis 단일 endpoint 연결 모델을 유지하고, 현재의 작고 예측 가능한 부하에서 용량을 직접 제어한다.
- 별도의 캐시 런타임, 연결 관리와 장애 관측이 필요하다.

### ElastiCache Serverless for Valkey

- 용량 계획 없이 자동 확장하고 다중 AZ 저장과 99.99% SLA를 기본 제공한다.
- 현재 워크로드는 작고 예측 가능해 자동 확장의 이득이 작고, cluster mode enabled 클라이언트만 지원해 단순한 단일 endpoint 운영 모델보다 제약이 크다.
- 운영 부담이 노드 용량 제어보다 중요해지거나 부하가 급격히 변하면 다시 비교한다.

### MemoryDB for Valkey

- 다중 AZ 트랜잭션 로그로 내구성을 제공하여 Redis-compatible 주 데이터베이스로 사용할 수 있다.
- 유실되면 다시 생성하는 파생 캐시에는 영속성과 비용을 정당화할 수 없어 채택하지 않았다.

### PostgreSQL 영구 저장

- 기존 RDS를 그대로 사용하고 재시작·축출에도 결과를 보존할 수 있다.
- 그러나 설명은 만료 후 재생성하려는 파생 데이터이며, TTL 정리와 기능별 파생 데이터가 영구 도메인 저장소에 누적된다.
- Redis를 별도 캐싱 계층으로 두려는 현재 방향과 맞지 않아 채택하지 않았다.

### PostgreSQL + Redis 2단계 저장

- 영구 보존과 빠른 조회를 함께 제공한다.
- 설명 유실을 허용하고 재생성할 수 있는 MVP에서 이중 쓰기·무효화와 운영 복잡도를 정당화하지 못해 채택하지 않았다.

### FE 메모리 캐시

- 구현 범위가 작지만 새로고침·재접속·다른 사용자와 Backend 인스턴스 사이에 결과를 공유하지 못한다.
- 같은 설명의 AI 호출과 비용이 반복되므로 채택하지 않았다.

### Unicode NFC 정규화 — 채택

- 조합형·분해형처럼 같은 문자의 서로 다른 Unicode 표현을 통일하면서 호환 문자와 과학·수학 기호의 표기는 보존한다.
- 같은 용어를 일부 놓치면 AI 생성이 한 번 더 발생하지만, 서로 다른 학술 기호에 잘못된 설명을 재사용하는 위험은 낮다.
- 논문 학습에서는 중복 생성 비용보다 설명 정확성을 우선하므로 채택했다.

### Unicode NFKC 정규화

- 전각 문자, 합자, 원문자와 일부 호환 기호까지 일반 문자로 통일해 캐시 중복을 더 줄일 수 있다.
- `²`와 `2`, `Ⅳ`와 `IV`처럼 논문에서 구분할 수 있는 표기까지 같은 문자열로 합쳐 잘못된 설명을 공유할 수 있어 채택하지 않았다.

## 5. Consequences

- local과 배포 환경에 Valkey-compatible cache와 Backend 연결 설정이 필요하다.
- Backend는 cache HIT·MISS, 생성 결과 저장, 고정 TTL, 사용자별 동시 생성 거절과 캐시 장애를 관측해야 한다.
- Valkey 캐시 데이터는 복구 대상인 원본 데이터가 아니다. 유실되면 다음 요청에서 AI로 재생성한다.
- DEV·PROD 각각 노드 1대로 시작하므로 노드 교체나 장애 동안 캐시 기능이 일시적으로 사용 불가할 수 있다. 해당 가용성이 제품 요구로 올라가면 replica 1대와 Multi-AZ 자동 failover를 추가한다.
- 인기 용어도 마지막 조회가 아니라 마지막 생성으로부터 30일 뒤 만료되어 새 설명으로 갱신된다.
- TTL 만료는 AI를 다시 실행할 기회를 제공할 뿐 최신 동향 반영 자체를 보장하지 않는다. 실제 최신 외부 정보가 필요하면 retrieval을 별도 기능으로 도입해야 한다.

## 6. Revisit Triggers

- cache MISS 재생성 비용이나 응답 지연이 허용 범위를 넘으면 사전 생성 또는 영구 저장을 다시 비교한다.
- 30일보다 짧거나 긴 갱신 주기가 필요하다는 사용 데이터가 쌓이면 기본 TTL을 조정한다.
- 서로 다른 Document 사이에서도 같은 개념 설명을 공유해야 하면 cache identity와 문맥 경계를 다시 설계한다.
- Valkey 장애 중 AI 호출 증폭을 운영 정책만으로 제어하기 어렵다면 실패 방식과 보조 저장소를 다시 검토한다.
- 소용량 node의 메모리·연결·처리량 지표가 임계치에 접근하면 node 크기를 늘리거나 Serverless를 다시 비교한다.

## 7. References

- [FT-012 선행지식](../features/FT-012-선행지식.md)
- [BE ↔ AI OpenAPI](../contracts/backend-ai/openapi.yml)
- [Amazon ElastiCache 배포 방식 비교](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.deployment.html)
- [Amazon ElastiCache 요금](https://aws.amazon.com/elasticache/pricing/)
