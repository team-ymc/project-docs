# ADR-008: ALB 인바운드는 CloudFront 관리형 prefix list로만 허용한다

## 1. Overview

- Date: 2026-09-02
- Status: Accepted
- Deciders: TBD
- Tracking: [YMC-303](https://geunhh.atlassian.net/browse/YMC-303)
- Related: [AWS Infrastructure](../architecture/aws-infrastructure.md)

## 2. Context

`/api/*` 요청은 CloudFront → ALB → Backend 경로로 전달된다. ALB는 public subnet의 internet-facing 로드밸런서이고 `api.<env>.papertutor.co.kr`이 공개 DNS로 ALB를 직접 가리키므로, CloudFront를 거치지 않고 ALB에 직접 요청할 수 있다. 현재 ALB 보안그룹 인바운드는 80·443 모두 `0.0.0.0/0`이다.

이 경로가 열려 있으면 CloudFront에 붙이는 WAF rate limit 등 CloudFront 층의 보안 요소가 ALB 직접 호출로 우회된다.

## 3. Decision

ALB 보안그룹의 443 인바운드를 `0.0.0.0/0` 대신 CloudFront 관리형 prefix list(`com.amazonaws.global.cloudfront.origin-facing`)로 제한한다. dev·prod 모두 적용한다.

80 인바운드 규칙은 제거한다. 
- CloudFront가 ALB로 나갈 때 HTTPS(443)만 쓰도록 설정되어 있어 80으로 오는 요청이 없다. 
- 보안그룹 하나에 넣을 수 있는 인바운드 규칙은 기본 60개인데, CloudFront prefix list를 참조하는 규칙 하나는 목록 용량인 55개로 계산된다.

헬스체크·디버깅용 직접 접근 경로는 두지 않는다. 배포 확인은 CloudFront 경유 `/api/*` 응답과 `aws elbv2 describe-target-health`로 한다. 디버깅에 직접 접근이 꼭 필요하면 출발지 IP `/32` 규칙을 일시 추가했다가 제거한다. 남는 규칙 여유는 5개다.

변경은 `infra/deploy/modules/foundation/main.tf`의 인바운드 규칙과 prefix list 조회 data source 하나다. CloudFront, ALB 리스너, 애플리케이션은 바꾸지 않는다.

CloudFront 대역이 아닌 출발지의 연결은 보안그룹에서 폐기되어 TCP 연결이 성립하지 않는다.

## 4. Options Considered

### CloudFront 관리형 prefix list로 보안그룹 제한 — 채택

- 규칙 변경으로 끝나고 차단이 TCP 연결 단계에서 일어나 ALB와 애플리케이션이 변경을 모른다.
- 다른 AWS 계정의 CloudFront도 같은 대역이므로 통과한다. 이 틈은 WAF와 애플리케이션 인증이 덮는다.

### CloudFront 커스텀 헤더를 ALB 리스너 룰에서 검사

- 우리 CloudFront만 통과시킬 수 있다.
- 헤더 값을 비밀로 관리하고 회전해야 하며, 차단이 HTTP 단계라 ALB까지는 연결이 닿는다. 지금 막으려는 위협은 직접 호출이므로 먼저 채택하지 않고 남용이 관측되면 추가한다.

### CloudFront VPC origins로 ALB를 internal로 전환

- ALB가 public IP를 갖지 않아 직접 접근 경로 자체가 사라지고, 공개 DNS `api.*`와 ALB 리스너용 ACM 인증서가 필요 없어진다.
- ALB를 internal·private subnet으로 재생성하고 CloudFront origin을 VPC origin으로 교체해야 하며, VPC origin에서도 보안그룹은 prefix list로 허용해야 한다. 막으려는 위협에 비해 마이그레이션 비용이 커 채택하지 않는다.

### 현행 유지

- WAF와 애플리케이션 인증·사용량 제한에만 의존한다.
- ALB로 직접 api를 호출할 경우 WAF가 우회되므로 채택하지 않는다.

## 5. Consequences

- 모든 외부 `/api/*` 트래픽이 CloudFront를 지나므로 CloudFront 층의 WAF 룰이 우회되지 않는다.
- `/api/*` 밖의 경로(`/actuator/*`, `/livez`, `/readyz`)는 외부에서 닿지 않는다. ALB → Task 헬스체크는 VPC 내부 경로라 영향이 없다.
- `api.<env>.papertutor.co.kr`로 직접 curl하는 런북 절차는 CloudFront 경유 확인과 target health 조회로 바꾼다.

## 6. Revisit Triggers

- 다른 계정의 CloudFront를 통한 우회가 실제 남용으로 관측되면 CloudFront 커스텀 헤더를 ALB 리스너 룰에서 검사하는 방식을 추가한다.
- ALB 보안그룹에 규칙이 5개 넘게 필요해지면 Service Quotas 상향을 요청한다.
- ALB를 public에 두지 않아야 하는 요구가 생기면 VPC origins를 재검토한다.

## 7. References

- [Use the CloudFront managed prefix list — Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/LocationsOfEdgeServers.html#managed-prefix-list)
- [AWS-managed prefix list weight — Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-aws-managed-prefix-lists.html#aws-managed-prefix-list-weights)
- [Restrict access with VPC origins — Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-vpc-origins.html)
