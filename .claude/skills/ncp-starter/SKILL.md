---
name: ncp-starter
description: NAVER Cloud Platform(NCP)에 백엔드용 Server(VM)를 만들고 그 위에 코드를 배포할 때 사용. 사용자가 "NCP에 서버 띄워줘", "네이버클라우드 VM 만들어줘", "백엔드 서버 하나 필요해", "NCP에서 Node/Python 서버 띄워야 해", "내 프론트에 붙일 API 서버 만들어줘" 같이 컴퓨팅 자원이 필요한 의도를 보일 때 자동 활성화. 모든 CLI 작업은 ncloud-cli로 수행하며, 자격증명은 NCLOUD_ACCESS_KEY/NCLOUD_SECRET_KEY 환경 변수에서 읽고, 리전은 KR 고정. 파일 저장이 추가로 필요할 때만 Object Storage를 보조로 다룬다.
---

# NCP Starter Skill (Server-first)

이 스킬은 NAVER Cloud Platform(NCP)에 Server(VM)를 만들고 백엔드 코드를 올리는 작업을 표준화한다.
사용자가 NCP 서버 관련 작업을 요청하면, 본 스킬의 규칙과 워크플로를 따라 진행하라.

## 1. 우선 확인할 것

사용자의 첫 요청을 받으면 다음 정보를 짧게 확인한다. 모르면 사용자에게 묻는다.

- 어떤 백엔드를 띄울 것인가 (Node Express, Python FastAPI, 정적 호스팅, 단순 webhook 등)
- 트래픽 규모와 가용성 기대치 (개인 프로젝트 / 데모 / 운영)
- 사용자의 NCP 사용 단계 (가입 직후 / 키만 있음 / 이미 VPC가 있음)

확인이 끝나면 어떤 NCP 자원을 만들지 한 줄로 제안하고 사용자의 동의를 받는다.

## 2. CLI 도구 원칙

모든 NCP 자원 조작은 **`ncloud-cli`** 만 사용한다. `aws cli` 를 NCP 자원에 사용하지 않는다.

```bash
ncloud vserver <command> --regionCode KR
```

설치·설정 방법은 `references/credentials.md` 참조.

## 3. 자격증명 규칙

자격증명은 다음 환경 변수에서만 읽는다. 사용자에게 키 값을 묻거나 코드에 박지 않는다.

- `NCLOUD_ACCESS_KEY`
- `NCLOUD_SECRET_KEY`
- `NCLOUD_API_URL` (기본 `https://ncloud.apigw.ntruss.com`)

`~/.ncloud/configure` 파일이 있으면 그쪽이 우선된다.

## 4. 표준 워크플로 (Server 생성)

사용자: "Node API 서버 하나 띄워 줘"

1. **사전 자원 확인**
   - VPC, Subnet, ACG(보안그룹), Login Key(SSH 키) 가 있는지 확인.
   - 없으면 `references/server-vm.md` 의 "콘솔 사전 준비" 절차를 안내하고, 사용자가 콘솔에서 만들 때까지 기다린다.
   - 자동 생성을 시도하지 않는다. 네트워크 자원은 실수 시 영향이 크므로 사용자의 명시적 클릭을 요구한다.

2. **사양 추천**
   - 기본값: 가장 저렴한 마이크로 등급, Ubuntu 22.04 LTS, 50GB 디스크.
   - 사용자에게 사양·OS·디스크 크기를 한 번에 확인받는다.

3. **서버 생성 명령 실행** (확인 후)
   - `ncloud vserver createServerInstances` 를 호출.
   - 필수 인자: `--vpcNo`, `--subnetNo`, `--serverImageProductCode`, `--serverProductCode`, `--loginKeyName`, `--networkInterfaceList`, `--serverName`.
   - 명령 실행 직전 모든 인자를 표로 정리해 사용자에게 한 번 확인받는다.

4. **상태 폴링**
   - `ncloud vserver getServerInstanceList` 로 상태를 확인.
   - `RUN` 상태가 되면 공인 IP를 사용자에게 알린다.

5. **접속 안내**
   - Login Key(.pem)로 SSH 접속하는 명령을 사용자에게 출력.
   - 비밀번호 방식이면 콘솔에서 비밀번호를 받아 오는 절차를 안내.

6. **백엔드 코드 배포 (사용자가 원할 때만)**
   - Node.js / Python 설치 → 코드 클론/업로드 → 의존성 설치 → 프로세스 매니저(pm2, systemd 등)로 실행.
   - 한 번에 한 단계씩 사용자 확인을 받으며 진행.

7. **포트 오픈**
   - 백엔드가 사용하는 포트(예: 80, 443, 3000)를 ACG 인바운드 규칙에 추가.
   - `ncloud vserver addNetworkInterfaceAccessControlGroup` 등 ACG 관련 명령은 변경 직전 확인을 한 번 더.

8. **프론트엔드 연결**
   - 사용자의 프론트 코드에서 백엔드 URL을 환경 변수로 받도록 안내.
   - HTTPS가 필요하면 도메인 + Let's Encrypt 또는 NCP 인증서 서비스 안내(2편 범위).

## 5. 안전 규칙

1. **리소스 생성** 명령(`createServerInstances`, `createVpc` 등) 직전에 이름·사양·VPC/Subnet 번호를 표로 정리해 한 번 확인받는다.
2. **파괴적 명령**(`terminateServerInstances`, `stopServerInstances`, `deleteVpc` 등)은 실행 전 반드시 한 번 더 확인받는다. "정말 진행할까요? (y/N)" 패턴.
3. 자격증명·계정 ID·공인 IP 전체는 화면에 그대로 출력하지 말고 마스킹한다 (예: `203.xx.xx.45`).
4. 본 스킬 범위(Server, ACG, 선택적 Object Storage)를 벗어난 자원(NKS, Cloud DB, Auto Scaling, Load Balancer 등)은 사용자가 명시적으로 요청한 경우에만 다룬다.

## 6. 작명 컨벤션

- 서버: `<프로젝트명>-<용도>` (예: `mysite-api`, `mysite-worker`)
- 동일 용도 다중: `<프로젝트명>-<용도>-<번호>` (예: `mysite-api-1`, `mysite-api-2`)
- Login Key: `<프로젝트명>-key`
- ACG: `<프로젝트명>-acg-<용도>` (예: `mysite-acg-web`)

상세는 `references/conventions.md` 참조.

## 7. 출력 규칙

- `ncloud-cli` 응답 JSON을 그대로 노출하지 말고, 핵심 필드만 표로 요약한다.
- 에러 발생 시 추측으로 재시도하지 않는다. 원인 가설 1~3개 + 검증 방법 + 다음 행동 제안.
- 후속 명령을 자동으로 묶어 실행하지 않는다. 단계마다 사용자 확인.

## 8. 선택적 보조: Object Storage

사용자의 시나리오가 파일 저장(이미지, 업로드 파일, 정적 자산 등)을 필요로 하는 경우에만 활성화한다. 그 경우:

- **버킷 관리(생성·삭제·목록)**: `ncloud objectstorageservice` 명령으로 수행.
- **파일 업로드/다운로드 등 데이터 조작**: NCP는 S3 호환 API를 제공하므로, 사용자 애플리케이션 코드에서는 `boto3`(Python) 또는 `@aws-sdk/client-s3`(JS/TS) 를 사용한다. 이는 NCP가 공식 문서에서 권장하는 방식이다.
- **CLI에서 파일 조작이 필요한 경우**: NCP 콘솔의 웹 UI를 사용하도록 안내한다.

상세 패턴은 `references/object-storage.md` 참조.

## 9. 범위 밖

- 멀티 리전 / 멀티 AZ 구성, Auto Scaling, Load Balancer, Cloud DB, NKS — 본 1편에서는 다루지 않는다.
- 사용자가 위 주제를 묻는다면 "후속 편에서 다룰 예정"으로 안내한다.

## References

필요할 때만 다음 파일을 읽어 더 상세한 정보를 활용하라.

- `references/credentials.md` — NCP 가입, API Key, ncloud-cli 설치·설정
- `references/server-vm.md` — VPC 사전 준비, 서버 생성, 접속, 배포 패턴 (Primary)
- `references/object-storage.md` — 파일 저장이 필요할 때만 (Secondary)
- `references/conventions.md` — 작명·태깅·환경변수 컨벤션 상세
