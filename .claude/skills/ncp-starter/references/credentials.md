# NCP 자격증명 설정 (ncloud-cli 기준)

이 문서는 NCP 신규 가입부터 ncloud-cli 설치·설정까지를 다룬다.
스킬 본문(SKILL.md)에서 자격증명이 필요할 때만 이 파일을 참조하라.

## 1. NCP 계정 만들기

1. <https://www.ncloud.com> 에 접속해 회원가입.
2. 결제 수단을 등록한다. 신규 가입 크레딧이 지급되어 본 스킬의 실습 범위는 무료로 진행된다.
3. 콘솔 우측 상단 사용자 메뉴 > **마이페이지 > 계정 관리 > 인증키 관리** 로 이동.

## 2. API Key 발급

1. **신규 API 인증키 생성** 버튼 클릭.
2. **Access Key ID** 와 **Secret Key** 두 값을 안전한 곳에 저장.
   - Secret Key는 한 번만 보인다. 즉시 보관.
3. (권장) **Sub Account** 기능으로 Server·VPC 권한만 부여한 별도 키를 발급해 두면, 실수로 다른 자원을 건드릴 위험을 줄인다.

## 3. ncloud-cli 설치

NCP 공식 CLI 도구다. 본 스킬에서 모든 자원 조작에 사용한다.

### macOS

```bash
brew tap navercloudplatform/ncloud-cli
brew install ncloud-cli
```

설치되지 않는다면 공식 가이드의 바이너리 다운로드 절차를 따른다.
공식 문서: <https://cli.ncloud-docs.com/docs/ko/home>

### Linux / Windows

공식 문서의 OS별 설치 절차를 따른다. JRE/JDK 8+ 가 필요할 수 있다.

### 설치 확인

```bash
ncloud --version
```

## 4. 자격증명 등록

두 가지 방법이 있다. 둘 중 하나만 하면 된다.

### 방법 A. 환경 변수 (간단)

`~/.zshrc` 또는 `~/.bashrc` 에 추가:

```bash
export NCLOUD_ACCESS_KEY="<발급받은 Access Key>"
export NCLOUD_SECRET_KEY="<발급받은 Secret Key>"
export NCLOUD_API_URL="https://ncloud.apigw.ntruss.com"
```

적용:

```bash
source ~/.zshrc   # 또는 ~/.bashrc
```

### 방법 B. configure 파일 (다중 프로필 가능)

```bash
mkdir -p ~/.ncloud
cat > ~/.ncloud/configure <<EOF
[DEFAULT]
ncloud_access_key_id = <발급받은 Access Key>
ncloud_secret_access_key = <발급받은 Secret Key>
ncloud_api_url = https://ncloud.apigw.ntruss.com
EOF

chmod 600 ~/.ncloud/configure
```

이 방식은 여러 프로필을 만들 수 있다(예: `[STAGING]`, `[PROD]`). 사용 시 `--configName STAGING` 같이 지정.

## 5. 동작 확인

```bash
ncloud vserver getRegionList
```

리전 목록(`KR`, `KRS`, `SGN` 등)이 표시되면 자격증명 설정 성공.

## 6. 키 노출 주의

- 자격증명을 코드(특히 프론트엔드 번들)에 직접 박지 않는다.
- `.env` 파일을 사용하고 `.gitignore` 에 `.env` 를 포함시킨다.
- 사용자가 키를 채팅에 붙여 넣은 경우, 응답에서 마스킹해서 보여 준다(예: `ABCD****`).
- Git에 실수로 키가 올라간 경우, 즉시 콘솔에서 키를 폐기·재발급하고 커밋 히스토리에서도 제거(`git filter-repo` 등).

## 7. 추가 환경 변수 (선택)

코드 안에서 NCP 자원에 접근할 때 사용할 추가 변수.

- `NCLOUD_REGION` — 기본 `KR`
- `NCP_BUCKET` — Object Storage를 함께 쓸 경우, 기본 버킷명
- `NCP_OBJECT_STORAGE_ENDPOINT` — `https://kr.object.ncloudstorage.com` (Object Storage SDK 사용 시)
