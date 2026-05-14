# NCP × Claude Starter

> NAVER Cloud Platform(NCP)에 백엔드 서버를 가장 빠르게 띄우기 위한 Claude Code Skill 패키지.
> 사용자가 "NCP에 서버 띄워줘", "백엔드 서버 하나 필요해" 처럼 말하면 자동으로 활성화됩니다.

---

## 무엇을 제공하나요

- 자연어 한 줄로 NCP Server(VM)를 만들고 그 위에 백엔드 코드를 배포할 수 있게 해 주는 **Claude Code Skill**
- 모든 NCP 자원 조작은 NCP 공식 CLI(**`ncloud-cli`**)로 수행
- 자격증명·작명·안전 규칙을 미리 정의한 컨텍스트 — 매번 같은 설명을 반복할 필요가 없고, **호출 토큰도 절약**됩니다
- (보조) Skill을 못 쓰는 환경(Claude Desktop, Cursor 등)을 위한 **프롬프트 팩** — `for-vibe-coders/`
- (보조) 파일 저장이 추가로 필요할 때 Object Storage 가이드 활성화

---

## 1. 설치 방법 (Claude Code 사용자)

### 방법 A. 프로젝트별 설치 (이 레포를 직접 사용)

```bash
git clone https://github.com/<your-name>/ncp-claude-starter.git
cd ncp-claude-starter
claude
```

Claude Code가 시작될 때 `.claude/skills/ncp-starter/` 를 자동 인식합니다. 다른 폴더에서는 활성화되지 않습니다.

### 방법 B. 글로벌 설치 (모든 프로젝트에서 사용)

```bash
mkdir -p ~/.claude/skills
cp -r .claude/skills/ncp-starter ~/.claude/skills/
```

이제 어느 디렉토리에서 `claude` 를 실행해도 활성화됩니다.

### 활성화 확인

Claude Code를 실행한 뒤 다음과 같이 물어보세요.

```
NCP에 Node API 서버 하나 띄우고 싶어. 어디서부터 시작하지?
```

Claude가 본 스킬의 규칙(VPC·Subnet·ACG·Login Key 사전 확인 → 사양 추천 → 사용자 확인 → ncloud-cli 호출)을 따라 응답하면 활성화 성공입니다.

---

## 2. 사전 준비

본 스킬을 처음 쓰기 전 한 번만 해 두면 됩니다.

1. <https://www.ncloud.com> 에서 회원가입 + 결제 수단 등록 (신규 가입 크레딧 자동 지급)
2. **마이페이지 > 계정 관리 > 인증키 관리**에서 Access Key / Secret Key 발급
3. **`ncloud-cli`** 설치

```bash
# macOS
brew tap navercloudplatform/ncloud-cli
brew install ncloud-cli

# 그 외 OS는 공식 가이드 참고
# https://cli.ncloud-docs.com/docs/ko/home
```

4. 자격증명 등록 (둘 중 하나)

**환경 변수**:

```bash
export NCLOUD_ACCESS_KEY="<발급받은 Access Key>"
export NCLOUD_SECRET_KEY="<발급받은 Secret Key>"
export NCLOUD_API_URL="https://ncloud.apigw.ntruss.com"
```

**configure 파일**:

```bash
mkdir -p ~/.ncloud
cat > ~/.ncloud/configure <<EOF
[DEFAULT]
ncloud_access_key_id = <Access Key>
ncloud_secret_access_key = <Secret Key>
ncloud_api_url = https://ncloud.apigw.ntruss.com
EOF
chmod 600 ~/.ncloud/configure
```

5. 동작 확인

```bash
ncloud vserver getRegionList
```

리전 목록(`KR`, `KRS`, `SGN` 등)이 보이면 성공.

자세한 절차는 `.claude/skills/ncp-starter/references/credentials.md` 참조.

---

## 3. 사용 예시

스킬이 설치되면 다음과 같이 자연어로 요청하기만 하면 됩니다.

```
내 Next.js 프론트에서 호출할 Express API 서버 하나 띄우고 싶어.
```

```
방금 만든 mysite-api 서버에 SSH로 접속할 수 있는 명령 알려 줘.
```

```
mysite-api 서버를 정리해 줘. 공인 IP도 같이 반납.
```

Claude가 자동으로 본 스킬의 컨벤션·안전 규칙을 따라 단계별로 진행합니다. (네트워크 자원은 콘솔에서 만들도록 안내, 생성·삭제 직전 확인 등)

---

## 4. 폴더 구조

```
ncp-claude-starter/
├── README.md                                    ← 지금 이 문서
├── .claude/
│   └── skills/
│       └── ncp-starter/
│           ├── SKILL.md                         ← 스킬 본체 (Server 중심 + 자동 활성화)
│           └── references/
│               ├── credentials.md               ← 가입·키·ncloud-cli 설치
│               ├── server-vm.md                 ← 서버 생성·접속·배포 (Primary)
│               ├── object-storage.md            ← 파일 저장 필요 시만 (Secondary)
│               └── conventions.md               ← 작명·태깅·환경 변수
└── for-vibe-coders/                             ← Skill 미지원 환경용 보조 경로
    ├── README.md
    └── prompt-pack.md
```

---

## 5. Skill 미지원 환경(Claude Desktop, Cursor 등)을 위한 보조 경로

Claude Code가 아닌 다른 AI 에디터를 쓴다면 `for-vibe-coders/README.md` 를 따라가세요. `prompt-pack.md` 의 시스템 프롬프트를 한 번 등록하면 비슷한 일관성을 얻을 수 있습니다.

---

## 6. 다음 편 예고

- **2편**: 도메인 + HTTPS 인증서(NCP Certificate Manager)로 운영 수준 마감
- **3편**: 폼 데이터·사용자 정보를 위한 Cloud DB for MySQL 연결
- **4편**: NCP 인프라용 미니 MCP 서버 자체 제작
- **5편**: Object Storage 단독 시나리오 (정적 호스팅·이미지 CDN)

피드백·요청은 Issues 탭에 자유롭게 남겨 주세요. 다음 편 우선순위에 반영됩니다.

---

## 참고

- [NCP 콘솔](https://www.ncloud.com/)
- [ncloud-cli 공식 문서](https://cli.ncloud-docs.com/docs/ko/home)
- [VPC Server 사용 가이드](https://guide.ncloud-docs.com/docs/server-server-1-2v2)
- [Anthropic Claude Code Skills 문서](https://docs.claude.com/en/docs/claude-code/skills)
