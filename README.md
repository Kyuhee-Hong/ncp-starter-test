# NCP × Claude Starter

> NAVER Cloud Platform(NCP)에 백엔드를 가장 빠르게 띄우기 위한 Claude Code Skill 패키지.
> 사용자가 "NCP에 백엔드 붙여줘", "object storage 버킷 만들어줘" 처럼 말하면 자동으로 활성화됩니다.

---

## 무엇을 제공하나요

- 자연어 한 줄로 NCP Object Storage·Server를 다룰 수 있게 해 주는 **Claude Code Skill**
- 자격증명·작명·안전 규칙을 미리 정의한 컨텍스트 — 매번 같은 설명을 반복할 필요가 없고, **호출 토큰도 절약**됩니다
- (보조) Skill을 못 쓰는 환경(Claude Desktop, Cursor 등)을 위한 **프롬프트 팩** — `for-vibe-coders/`

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
NCP에 파일 업로드 버킷 하나 만들고 싶어.
```

Claude가 본 스킬의 규칙(자격증명 위치 확인, 버킷 이름 컨벤션, 생성 전 확인 등)을 따라 진행하면 활성화 성공입니다.

---

## 2. 사전 준비

본 스킬을 처음 쓰기 전 한 번만 해 두면 됩니다.

1. <https://www.ncloud.com> 에서 회원가입 + 결제 수단 등록 (신규 가입 크레딧 자동 지급)
2. **마이페이지 > 계정 관리 > 인증키 관리**에서 Access Key / Secret Key 발급
3. 환경 변수 등록 (`~/.zshrc` 또는 `~/.bashrc`)

```bash
export NCP_ACCESS_KEY="<발급받은 Access Key>"
export NCP_SECRET_KEY="<발급받은 Secret Key>"
export NCP_REGION="kr-standard"
export NCP_ENDPOINT="https://kr.object.ncloudstorage.com"
```

4. AWS CLI 설치 + 프로필 등록 (NCP Object Storage가 S3 호환)

```bash
brew install awscli            # macOS, Linux는 apt 등
aws configure --profile ncp
```

자세한 절차는 `.claude/skills/ncp-starter/references/credentials.md` 참조.

---

## 3. 사용 예시

스킬이 설치되면 다음과 같이 자연어로 요청하기만 하면 됩니다.

```
내 Next.js 프로젝트에 NCP Object Storage로 파일 업로드 기능을 붙여 줘.
```

```
ncp-uploads-a8f3 버킷 안에 있는 파일 목록을 보여 줘.
```

```
방금 만든 버킷을 비우고 삭제해 줘.
```

Claude가 자동으로 본 스킬의 컨벤션·안전 규칙을 따라 단계별로 진행합니다.

---

## 4. 폴더 구조

```
ncp-claude-starter/
├── README.md                                  ← 지금 이 문서
├── .claude/
│   └── skills/
│       └── ncp-starter/
│           ├── SKILL.md                       ← 스킬 본체 (자동 활성화 트리거 + 규칙)
│           └── references/
│               ├── credentials.md             ← 가입·키 발급·환경 변수
│               ├── object-storage.md          ← 버킷·파일 패턴(JS·Python·CLI)
│               └── conventions.md             ← 작명·태깅·파일 키 컨벤션
└── for-vibe-coders/                           ← Skill 미지원 환경용 보조 경로
    ├── README.md
    └── prompt-pack.md
```

---

## 5. Skill 미지원 환경(Claude Desktop, Cursor 등)을 위한 보조 경로

Claude Code가 아닌 다른 AI 에디터를 쓴다면 `for-vibe-coders/README.md` 를 따라가세요. `prompt-pack.md` 의 시스템 프롬프트를 한 번 등록하면 비슷한 일관성을 얻을 수 있습니다.

---

## 6. 다음 편 예고

- **2편**: Server(VM) + Express 기반 미니 업로드 API
- **3편**: 작은 데이터(폼·방명록)를 위한 Cloud DB for MySQL 연결
- **4편**: NCP 인프라용 미니 MCP 서버 자체 제작

피드백·요청은 Issues 탭에 자유롭게 남겨 주세요.

---

## 참고

- [NCP 콘솔](https://www.ncloud.com/)
- [Object Storage 시작 가이드](https://guide.ncloud-docs.com/docs/objectstorage-start)
- [Anthropic Claude Code Skills 문서](https://docs.claude.com/en/docs/claude-code/skills)
