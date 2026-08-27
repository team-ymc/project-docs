# Architecture

Paper Teacher 시스템의 실행 환경, 인프라 구성과 배포 흐름을 관리한다.

## Documents

| 문서 | 역할 |
|---|---|
| [AWS Infrastructure](aws-infrastructure.md) | AWS 구성, 컴포넌트 책임과 주요 통신 경로 |
| [CI/CD](ci-cd.md) | 환경별 배포 흐름, 배포 권한과 Rollback 전략 |
| [Runtime Timeouts](runtime-timeouts.md) | 요청 경로별 timeout/deadline 값과 대소 관계의 SSOT |

`assets/`는 Architecture 문서에서 참조하는 시각 자료를 보관한다.

## Related Documentation

- API와 메시지 형식: [Contracts](../contracts/README.md)
- 설계 선택과 근거: [Decision Records](../decisions/README.md)
- 기능 요구사항: [Features](../features/README.md)
- 팀 운영과 배포 관련 규칙: [Conventions](../conventions/README.md)

## Scope

- Architecture 문서는 현재 채택된 구조와 운영 방식을 설명한다.
- 선택 근거는 ADR을, 서비스 간 schema는 Contracts를 링크한다.
