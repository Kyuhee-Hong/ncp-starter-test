# 작명 · 태깅 · 파일 키 컨벤션

본 스킬을 따라 만들어지는 모든 NCP 리소스는 다음 컨벤션을 따른다.

## 1. 버킷 이름

`<프로젝트명>-<용도>-<랜덤4자리>` 형식.

- 모두 소문자, 하이픈 구분
- 마침표(`.`) 사용 금지 (정적 호스팅 도메인과 충돌 가능)
- 랜덤 4자리는 `openssl rand -hex 2` 로 생성

예시:
- `mysite-uploads-a8f3`
- `acme-static-3c91`
- `blog-images-7d04`

## 2. 서버(VM) 이름

`<프로젝트명>-<용도>` 형식.

- 용도는 짧고 의미가 분명한 단어 하나(`api`, `web`, `worker`, `db` 등)
- 동일 용도가 여러 대 필요한 경우 끝에 숫자를 붙임 (`mysite-api-1`, `mysite-api-2`)

## 3. 태그

생성하는 모든 리소스에 다음 태그를 추가하라.

| 키 | 값 예시 | 용도 |
| --- | --- | --- |
| `project` | `mysite` | 비용·자원 그룹핑 |
| `env` | `dev` / `prod` | 환경 구분 |
| `owner` | `<이메일 또는 핸들>` | 책임자 식별 |

## 4. 파일 키(객체 키)

업로드되는 파일의 키는 다음 패턴을 권장한다.

- 충돌 방지: `${Date.now()}-${원본파일명}` 또는 UUID 접두어
- 도메인 분리: `uploads/`, `thumbnails/`, `exports/` 같은 prefix
- 결과 예: `uploads/1715600000000-photo.png`

## 5. 환경 변수 명명

코드에서 NCP 자격증명을 읽을 때는 항상 다음 이름을 쓴다.

- `NCP_ACCESS_KEY`
- `NCP_SECRET_KEY`
- `NCP_REGION` (선택, 기본 `kr-standard`)
- `NCP_ENDPOINT` (선택, 기본 `https://kr.object.ncloudstorage.com`)
- `NCP_BUCKET` (프로젝트의 기본 버킷명)

`AWS_ACCESS_KEY_ID` 같은 표준 AWS 변수명을 그대로 쓰면, 실제 AWS 자격증명과 혼동될 수 있으므로 사용하지 않는다.
