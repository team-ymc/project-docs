# Paper Teacher Project Docs

Paper Teacher의 제품 정의, 사용자 경험, 기능 명세, 시스템 계약과 기술적 의사결정을 관리한다.

## Documents

| 영역 | 책임 | 시작 문서 |
|---|---|---|
| Product Brief | 제품 문제와 방향 | [Product Brief](project-brief/project-brief.md) |
| Features | 기능과 완료 조건 | [Features](features/README.md) |
| Design | 화면 구조, 상태와 디자인 시스템 | [Design](design/README.md) |
| Architecture | 인프라, 시스템 구성과 배포 구조 | [Architecture](architecture/README.md) |
| Contracts | FE·BE·AI 사이 API와 메시지 계약 | [Contracts](contracts/README.md) |
| Decisions | 기술적 의사결정과 선택 근거 | [Decision Records](decisions/README.md) |
| Conventions | Git, Jira와 Sprint 운영 규칙 | [Conventions](conventions/README.md) |
| Deprecated | 더 이상 유지하지 않는 문서 | [Deprecated](deprecated/README.md) |

## Recommended Reading Order

`Product Brief → Features → Design`

## Document Identifiers

| Prefix | 의미 | Registry |
|---|---|---|
| `FT-XXX` | Feature | [Feature Registry](features/feature-spec.md#3-feature-registry) |
| `ADR-XXX` | Architecture Decision Record | [ADR Registry](decisions/README.md) |

새 문서는 해당 Registry에 먼저 등록하고 할당된 식별자를 파일명과 문서 제목에 동일하게 사용한다. `UF-XXX`, `WF-XXX`는 더 이상 발급하지 않는다([Deprecated](deprecated/README.md)).
