# Features

Paper Teacher가 제공해야 하는 동작을 독립적으로 개발·검증 가능한 Feature 단위로 관리한다.

## Documents

| 문서 | 역할 |
|---|---|
| [Feature Spec](feature-spec.md) | Feature Registry, 의존 관계와 상태 |
| [Feature 작성 양식](features_format.md) | `FT-XXX` 문서 구조와 작성 기준 |
| `FT-XXX-*.md` | Feature별 범위, Story, 완료 조건과 Open Questions |

## Related Documentation

- 화면 구조와 상태: [Design](../design/README.md)
- API와 메시지 계약: [Contracts](../contracts/README.md)
- 기술적 결정 근거: [Decision Records](../decisions/README.md)

## Adding a Feature

- 새 Feature는 Feature Registry에 먼저 등록하고 다음 `FT-XXX` ID를 할당한다.
- 화면이 있는 Feature는 [Design v2](../design/v2/README.md) 아트보드를 근거로 Story를 쓴다.
- [Feature 작성 양식](features_format.md)에 따라 개별 문서를 작성한다.
