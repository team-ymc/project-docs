# ADR-009: Backend 데이터베이스 구조 변경은 Flyway로 관리

## 1. Overview

- Date: 2026-09-16
- Status: Accepted
- Deciders: 근흐흐
- Tracking: TBD
- Related: [AWS Infrastructure](../architecture/aws-infrastructure.md), [CI/CD](../architecture/ci-cd.md)

## 2. Context

현재 local·dev는 Hibernate `ddl-auto: update`, prod는 `ddl-auto: validate`와 수동 SQL을 사용한다. 환경마다 데이터베이스 구조를 만드는 방법이 달라 변경 누락과 환경별 구조 차이를 확인하기 어렵다.

prod 배포 중에는 기존 버전과 새 버전의 Backend Task가 같은 데이터베이스를 함께 사용하므로, 데이터베이스 변경은 두 버전 모두와 호환되어야 한다.

## 3. Decision

Backend가 소유하는 PostgreSQL 구조는 Flyway로 관리한다.

- local·dev·prod와 테스트에 같은 변경 SQL을 순서대로 적용한다.
- Hibernate는 모든 환경에서 `ddl-auto: validate`만 사용한다.
- 적용 이력은 `flyway_schema_history` 테이블에 기록하며, 한 번 적용한 변경 SQL은 수정하지 않는다.
- 여러 Backend Task가 동시에 시작하면 Flyway의 데이터베이스 잠금으로 한 Task만 변경 SQL을 실행한다.
- 새 컬럼은 기존 Task와 기존 데이터를 고려해 nullable 또는 안전한 default로 추가하고, 새 코드는 값이 없는 경우를 처리한다.
- 컬럼 삭제·이름 변경·호환되지 않는 타입 변경은 기존 Task가 모두 내려간 뒤 후속 배포에서 수행한다.
- 실행 시간이 길거나 운영 부하가 큰 변경은 애플리케이션 시작 과정에서 분리한다.
- LangGraph checkpoint 테이블은 이번 결정의 범위에서 제외한다.

## 4. Options Considered

### Flyway — 채택

- 기존 PostgreSQL SQL을 그대로 활용할 수 있고 Spring Boot 연동이 단순하다.
- 현재 필요한 변경 순서와 적용 이력 관리를 가장 적은 복잡성으로 제공한다.

### Liquibase

- SQL 외에 YAML·XML·JSON과 세밀한 실행 조건을 지원한다.
- PostgreSQL 하나와 직접 작성한 SQL을 사용하는 현재 요구에는 관리할 개념이 더 많아 채택하지 않는다.


## 5. Consequences

- 모든 Backend 데이터베이스 구조 변경에는 버전이 붙은 Flyway SQL이 필요하다.
- JPA entity만 변경해서는 데이터베이스 구조가 바뀌지 않는다.
- 배포 전 빈 PostgreSQL 적용과 기존 데이터베이스의 추가 변경 적용을 모두 검증한다.
- 기존 `app/be/docs/db/*.sql`은 Flyway로 이전한 뒤 데이터베이스 구조의 기준 역할을 종료한다.

## 6. Revisit Triggers

- PostgreSQL 외의 데이터베이스를 함께 지원하거나 복잡한 환경별 실행 조건이 필요하면 Liquibase를 다시 비교한다.
- 변경 SQL이 애플리케이션 기동이나 운영 트래픽에 영향을 주면 별도의 일회성 변경 Task로 분리한다.

## 7. References

- [Flyway Getting Started](https://documentation.red-gate.com/flyway/getting-started-with-flyway)
- [Flyway Schema History Table](https://documentation.red-gate.com/fd/flyway-schema-history-table-273973417.html)
- [Spring Boot Database Initialization](https://docs.spring.io/spring-boot/how-to/data-initialization.html)
- [Liquibase Changelog](https://docs.liquibase.com/concepts/changelogs/home.html)
