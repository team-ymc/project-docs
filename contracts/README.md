# Contracts

FE·BE·AI 사이 계약의 SSOT 디렉토리

구조는 아래와 같다.

```
contracts/
├── README.md
├── openapi.yaml                       동기  FE ↔ BE (REST)            0.1.0 확정
├── asyncapi.yaml                      비동기 배선 — 큐 이름·방향       확정
└── schema/                            비동기 payload
    ├── parse-request.schema.json      BE → AI                        확정
    └── parse-result.schema.json       AI → BE                        envelope만 확정
```

`asyncapi.yaml`은 큐 이름·방향의 **SSOT**다. 큐 구현체(SQS 등)는 미결이라 `servers`가 아직 없다 — 정해지면 리전·버킷 같은 배선 값을 여기 두고 `infra/local/bootstrap.sh`와 `app/be/application.yml`이 따라간다.

## 규칙

- 계약이 코드보다 앞선다. 스키마를 바꿔야 하면 여기부터 PR로 고치고 코드가 따라간다.
- 참조할 계약이 없으면 코드에서 임의로 정의하지 말고 여기에 먼저 만든다.
- 한 사실은 한 곳에만 쓴다. 다른 문서는 링크만 건다.

## 버전

`0.x` 동안은 breaking change를 minor로 올린다 (0.1.0 → 0.2.0). 호환되는 필드 추가·설명 보완은 patch (0.1.1).
첫 안정 계약이 나오면 1.0.0으로 올린다.

## 소유권

- `PaperStatus`의 단일 writer는 BE다. 파싱 서버는 결과만 발행하고 상태를 직접 쓰지 않는다. (ADR-001)
- `parse-result`의 envelope(`paperId`·`status`·`error`)는 BE 소유, `result` 본문은 **AI 소유**다.

## 미확정

- **`parse-result.result`** — 비어 있다. 추측으로 채우지 말 것. BE 상태 전이엔 불필요하며, FT-004 Reader 착수 전까지 정의한다.
- **큐 구현체** — ADR-001의 옵션 항목.
