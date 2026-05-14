# 작명 · 태깅 · 환경 변수 컨벤션

본 스킬을 따라 만들어지는 모든 NCP 리소스는 다음 컨벤션을 따른다.

## 1. 서버(VM) 이름

`<프로젝트명>-<용도>` 형식.

- 모두 소문자, 하이픈 구분
- 용도는 짧고 의미가 분명한 단어 하나(`api`, `web`, `worker`, `db`, `bot` 등)
- 동일 용도가 여러 대 필요한 경우 끝에 숫자: `mysite-api-1`, `mysite-api-2`

예시:
- `mysite-api`
- `acme-worker`
- `blog-web-1`

## 2. Login Key 이름

`<프로젝트명>-key` 형식.

- 한 프로젝트의 모든 서버가 같은 Login Key를 공유해도 무방.
- `.pem` 파일은 발급 즉시 안전한 곳에 백업.
- `chmod 600` 으로 권한을 좁힐 것.

## 3. ACG(보안그룹) 이름

`<프로젝트명>-acg-<용도>` 형식.

- `mysite-acg-web` — HTTP/HTTPS 인바운드용
- `mysite-acg-internal` — 내부 통신용
- `mysite-acg-admin` — SSH 등 관리용 (소스 CIDR 좁게)

## 4. VPC / Subnet 이름

`<프로젝트명>-vpc`, `<프로젝트명>-subnet-<용도>` 형식.

- `mysite-vpc`
- `mysite-subnet-public` (퍼블릭 서버 배치용)
- `mysite-subnet-private` (DB 등 내부 자원용)

## 5. 태그

생성하는 모든 리소스에 다음 태그를 추가하라.

| 키 | 값 예시 | 용도 |
| --- | --- | --- |
| `project` | `mysite` | 비용·자원 그룹핑 |
| `env` | `dev` / `prod` | 환경 구분 |
| `owner` | `<이메일 또는 핸들>` | 책임자 식별 |

태그는 NCP 콘솔의 자원 페이지에서 추가하거나, 일부 CLI 명령의 인자로 지정할 수 있다.

## 6. 환경 변수 명명 (코드 안에서 NCP 접근 시)

| 변수명 | 용도 |
| --- | --- |
| `NCLOUD_ACCESS_KEY` | NCP Access Key |
| `NCLOUD_SECRET_KEY` | NCP Secret Key |
| `NCLOUD_API_URL` | API Gateway URL (기본 `https://ncloud.apigw.ntruss.com`) |
| `NCLOUD_REGION` | 리전 코드 (기본 `KR`) |
| `NCP_BUCKET` | (Object Storage 사용 시) 기본 버킷명 |
| `NCP_OBJECT_STORAGE_ENDPOINT` | (Object Storage 사용 시) `https://kr.object.ncloudstorage.com` |

> `AWS_ACCESS_KEY_ID` 같은 표준 AWS 변수명을 그대로 쓰면 실제 AWS 자격증명과 혼동될 수 있다. NCP 자원 접근은 항상 `NCLOUD_*` 접두어를 사용한다.

## 7. (선택) Object Storage 파일 키 컨벤션

Server 시나리오에 Object Storage가 보조로 들어갈 때, 객체 키는 다음을 따른다.

- 충돌 방지: `${Date.now()}-${원본파일명}` 또는 UUID 접두어
- 도메인 분리: `uploads/`, `thumbnails/`, `exports/` prefix
- 예: `uploads/1715600000000-photo.png`
