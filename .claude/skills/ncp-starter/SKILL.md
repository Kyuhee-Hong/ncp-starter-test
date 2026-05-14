---
name: ncp-starter
description: NAVER Cloud Platform(NCP) 위에 백엔드 인프라(Object Storage, Server/VM)를 빠르게 만들고 다룰 때 사용. 사용자가 "NCP에 백엔드 붙여줘", "네이버 클라우드 파일 저장소", "object storage 버킷 만들어줘", "S3 호환 스토리지", "네이버클라우드 서버 띄워줘" 같이 NCP 자원을 생성·조작하려는 의도를 보일 때 자동 활성화. AWS S3 호환 API(aws cli, boto3, @aws-sdk/client-s3)와 ncloud-cli를 활용하며, 자격증명은 환경 변수에서 읽고 리전은 kr-standard 고정.
---

# NCP Starter Skill

이 스킬은 NAVER Cloud Platform(NCP)에 백엔드 자원을 만들고 다루는 작업을 표준화한다.
사용자가 NCP 관련 작업을 요청하면, 본 스킬의 규칙과 워크플로를 따라 진행하라.

## 1. 우선 확인할 것

사용자의 첫 요청을 받으면 다음 정보를 짧게 확인한다.

- 만들려는 결과물의 형태(파일 저장소만 / API 서버까지 / 정적 호스팅 등)
- 사용 중인 프론트엔드 환경(React, Next.js, 정적 HTML, 없음 등)
- NCP 신규 사용자 여부 (Yes면 가입·키 발급 안내부터)

확인이 끝나면 어떤 NCP 자원을 만들지 한 줄로 제안하고 사용자의 동의를 받는다.

## 2. 자격증명 규칙

자격증명은 다음 환경 변수에서만 읽는다. 사용자에게 키 값을 묻거나 코드에 박지 않는다.

- `NCP_ACCESS_KEY`
- `NCP_SECRET_KEY`
- `NCP_REGION` (기본값 `kr-standard`)
- `NCP_ENDPOINT` (기본값 `https://kr.object.ncloudstorage.com`)

AWS CLI 프로필명은 `ncp`로 고정한다. 자세한 설정 방법은 `references/credentials.md`를 참조.

## 3. 명령 컨벤션

### Object Storage (S3 호환)

```bash
aws --profile ncp --endpoint-url "$NCP_ENDPOINT" s3 <subcommand>
```

- 버킷 생성 시 `--region kr-standard` 명시
- 코드(JS/TS)에서는 `@aws-sdk/client-s3` 사용, `endpoint: "https://kr.object.ncloudstorage.com"`, `region: "kr-standard"`

자세한 패턴은 `references/object-storage.md`를 참조.

### Server / 기타 NCP 자원

```bash
ncloud vserver <command> --regionCode KR
```

## 4. 안전 규칙

리소스 운영 중 다음을 반드시 지킨다.

1. 리소스 **생성**(`s3 mb`, `createServerInstances` 등) 직전에 이름·사양을 한 번 확인받는다.
2. 리소스 **삭제·정지·재시작** 같은 파괴적 명령은 실행 전 반드시 한 번 더 확인받는다.
3. 자격증명·서비스 URL·계정 ID는 화면에 그대로 출력하지 않고 마스킹한다.
4. 본 스킬 범위(Object Storage, Server)를 벗어난 자원(NKS, Cloud DB, VPC 세부 설정 등)은 사용자가 명시적으로 요청한 경우에만 다룬다.

## 5. 작명 컨벤션

- 버킷: `<프로젝트명>-<용도>-<랜덤4자리>` (예: `mysite-uploads-a8f3`)
- 서버: `<프로젝트명>-<용도>` (예: `mysite-api`)
- 태그: `project=<프로젝트명>`

## 6. 출력 규칙

- 명령 실행 결과의 JSON을 그대로 노출하지 말고, 표 또는 핵심 필드만 요약해서 보여 준다.
- 에러 발생 시 추측으로 재시도하지 않는다. 원인 가설 1~3개 + 다음 행동 제안을 먼저 한다.
- 후속 명령을 자동으로 묶어 실행하지 않는다. 사용자에게 각 단계의 확인을 받는다.

## 7. 워크플로 예시

사용자: "내 사이트에 파일 업로드 기능 붙여 줘"

1. 프론트엔드 환경 확인 → 응답에 따라 SDK 선택
2. NCP 가입·키 발급 여부 확인 → 미가입이면 `references/credentials.md` 절차 안내
3. 버킷 이름 제안 → 사용자 확인 → `aws s3 mb` 실행
4. 프론트엔드 코드에 업로드 핸들러 추가 (서버 라우트 경유, 키 노출 금지)
5. 동작 확인 절차 안내 → 비용·정리 방법 마지막에 안내

## 8. 범위 밖

- 멀티 리전, IAM 세부 정책, 운영 모니터링은 본 스킬에서 다루지 않는다.
- 사용자가 위 주제를 묻는다면 "후속 편에서 다룰 예정"으로 안내한다.

## References

필요할 때 다음 파일을 읽어 더 상세한 정보를 활용하라. 매번 전부 읽지 말고 그 작업에 필요한 것만.

- `references/credentials.md` — NCP 가입, API Key 발급, 환경 변수 설정
- `references/object-storage.md` — 버킷 생성·업로드·다운로드·삭제·CORS 패턴
- `references/conventions.md` — 작명·태깅·파일 키 컨벤션 상세
