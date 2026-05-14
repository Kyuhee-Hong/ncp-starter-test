# NCP 자격증명 설정

이 문서는 NCP 신규 가입부터 환경 변수 등록까지를 다룬다.
스킬 본문(SKILL.md)에서 자격증명이 필요할 때만 이 파일을 참조하라.

## 1. NCP 계정 만들기

1. <https://www.ncloud.com> 에 접속해 회원가입.
2. 결제 수단을 등록한다. 신규 가입 크레딧이 지급되어 본 스킬의 실습 범위는 무료로 진행된다.
3. 콘솔 우측 상단의 사용자 메뉴에서 **마이페이지 > 계정 관리 > 인증키 관리** 로 이동.

## 2. API Key 발급

1. **신규 API 인증키 생성** 버튼을 클릭.
2. 표시되는 **Access Key ID** 와 **Secret Key** 두 값을 안전한 곳에 저장.
   - Secret Key는 한 번만 보인다. 즉시 보관할 것.
3. (권장) **Sub Account** 기능으로 Object Storage·Server 권한만 부여한 별도 키를 발급해 두면, 실수로 다른 자원을 건드릴 위험을 줄일 수 있다.

## 3. 환경 변수 등록

사용자에게 다음 줄들을 셸 프로필(`~/.zshrc` 또는 `~/.bashrc`)에 추가하도록 안내한다.

```bash
export NCP_ACCESS_KEY="<발급받은 Access Key>"
export NCP_SECRET_KEY="<발급받은 Secret Key>"
export NCP_REGION="kr-standard"
export NCP_ENDPOINT="https://kr.object.ncloudstorage.com"
```

적용:

```bash
source ~/.zshrc   # 또는 ~/.bashrc
```

## 4. AWS CLI 프로필 등록

```bash
aws configure --profile ncp
# Access Key ID    : <NCP_ACCESS_KEY 값>
# Secret Key       : <NCP_SECRET_KEY 값>
# Default region   : kr-standard
# Default output   : json
```

이후 모든 명령은 `aws --profile ncp --endpoint-url "$NCP_ENDPOINT" ...` 형태로 호출한다.

## 5. 키 노출 주의

- 자격증명을 코드(특히 프론트엔드 번들)에 직접 박지 않는다.
- `.env` 파일을 사용하고, `.gitignore`에 `.env`가 포함되어 있는지 확인한다.
- 사용자가 키를 채팅에 붙여 넣으면, 응답에서 마스킹해서 보여 준다 (예: `AKIA****`).
