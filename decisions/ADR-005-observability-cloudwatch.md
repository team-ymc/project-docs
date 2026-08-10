# ADR-005: 로그와 메트릭은 CloudWatch에서 관리하고, 앱 로그는 30일 보존한다

## 1. Overview

- Date: 2026-07-25
- Status: Proposed
- Deciders: 근흐흐
- Tracking: YMC-252
- Implements: -
- Related: ADR-004, ADR-006(계측 규약)
- Supersedes / Superseded by: -

## 2. Context

BE(Spring Boot)와 AI(FastAPI)의 앱 로그를 한곳에서 조회하고, 서비스와 AWS 리소스의 상태를 같은 도구에서 확인할 수 있어야 한다.

팀은 2명이고 인프라 전담자가 없다. 초기 로그량은 일 10MB 안팎으로 예상하지만 아직 실측하지 않았다. 따라서 직접 운영하는 로그·메트릭 저장소를 두지 않고, AWS에서 이미 사용 중인 관리형 서비스로 시작한다.

## 3. Decision

### 3.1 로그

- 앱은 ADR-006에 따라 구조화 JSON 로그를 표준 출력으로 남긴다.
- ECS에서는 FireLens(Fluent Bit), EKS 전환 후에는 Fluent Bit DaemonSet이 앱 로그를 CloudWatch Logs로 보낸다.
- 앱 로그 그룹의 보존기간은 **30일**로 설정한다.
- 앱 로그를 S3에 별도로 복제하거나 아카이브하는 파이프라인은 만들지 않는다.
- 로그 그룹과 보존기간은 Terraform으로 관리한다.

30일보다 오래된 앱 로그가 실제 장애 분석이나 운영 지표 비교에 필요해지면 그때 보존기간을 늘린다. 변경은 Terraform의 보존기간 값만 수정하며, 이미 만료된 로그는 복구할 수 없음을 감수한다.

### 3.2 메트릭

- ECS·ALB·SQS·RDS가 기본 제공하는 메트릭은 각 AWS 네임스페이스에서 그대로 사용한다.
- BE와 AI는 ADR-006에 따라 `/metrics`에 Prometheus 포맷의 앱 메트릭을 노출한다.
- 현재는 앱 메트릭을 별도로 수집하지 않는다. 대시보드와 알람은 AWS 기본 메트릭으로 시작한다.
- 구체적인 앱 메트릭이 필요해지면 수집 방식, 대상과 라벨 규칙을 정한다.

### 3.3 비용 확인

현재 추정량은 CloudWatch 무료 사용량 안에 들어갈 것으로 예상한다. 이는 **CloudWatch 서비스 사용료에 대한 추정**이며, S3·ECS 실행 자원 등 다른 비용은 포함하지 않는다.

추정값을 결정 근거로 굳히지 않는다. 배포 첫 주에 다음 값을 확인해 이 ADR의 Updates에 기록한다.

- 앱 로그 그룹별 `IncomingBytes`
- 실제 CloudWatch 청구액

월 예상 비용이 $30을 넘으면 알림을 받도록 AWS Budget을 설정한다.

## 4. Options Considered

### Option A. CloudWatch Logs + AWS 기본 메트릭 — 채택

- 이미 사용하는 AWS 계정, IAM, ECS와 자연스럽게 연결된다.
- 로그 저장소, 대시보드, 알람 서버를 직접 운영하지 않아도 된다.
- 현재 팀 규모와 예상 사용량에 가장 단순하다.

### Option B. OpenSearch 또는 자체 호스팅 스택

- 검색 기능은 강하지만 인덱스와 클러스터 운영이 추가된다.
- 로그 검색이 팀의 일상 업무가 될 정도로 늘어나기 전에는 도입하지 않는다.

## 5. Consequences

### Positive

- 앱 로그와 AWS 기본 메트릭 조회·알람을 CloudWatch에서 시작할 수 있다.
- 팀이 별도 관측성 클러스터를 운영하지 않아도 된다.

### Trade-offs

- 앱 전용 메트릭의 수집과 알람은 아직 제공하지 않는다.
- 앱 로그는 30일이 지나면 삭제되며 복구할 수 없다. 더 긴 조회 기간이 필요해지면 보존기간을 늘려야 한다.
- Logs Insights의 검색·분석 기능은 전문 로그 분석 제품보다 제한적이다.

### Follow-ups

- ADR-006과 `conventions/logging.md`를 함께 확정한다.
- FireLens/Fluent Bit, 로그 그룹, AWS 기본 메트릭 대시보드와 알람을 Terraform으로 구현한다.
- 앱 전용 메트릭이 필요해지면 수집기, 수집 대상과 라벨 규칙을 정한다.
- 배포 첫 주 실측값을 Updates에 기록한다.

감사 로그(CloudTrail), ALB 액세스 로그와 장기 보관 정책은 이 ADR의 범위에 포함하지 않는다.

## 6. Updates

- (없음)
