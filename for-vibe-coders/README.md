# [보조 경로] Claude Code Skill을 못 쓰는 환경을 위한 프롬프트 팩

> 본 폴더는 Claude Desktop · Cursor · Lovable · v0 · ChatGPT 등 **Skill 자동 활성화가 지원되지 않는 AI 에디터** 사용자를 위한 보조 경로입니다.
> Claude Code 사용자라면 본 폴더를 무시하고 최상위 README의 안내를 따라 Skill을 설치하세요.

---

## 왜 별도 경로가 필요한가요

`.claude/skills/ncp-starter/` 폴더의 Skill은 Claude Code 환경에서만 자동 인식·자동 활성화됩니다.
다른 AI 에디터에서는 같은 규칙을 **시스템 프롬프트**로 등록해서 비슷한 효과를 얻을 수 있습니다.

---

## 시작하기

### 1. NCP 계정과 API Key

최상위 README의 "사전 준비" 섹션을 그대로 따라 진행합니다.
가입 → 인증키 발급 → 안전한 곳에 메모.

### 2. 시스템 프롬프트 등록

[`prompt-pack.md`](./prompt-pack.md) 파일의 "0. 시스템 프롬프트" 박스를 통째로 복사한 뒤, 자신의 AI 에디터에 등록합니다.

- **Claude Desktop**: Projects > 새 프로젝트 > Custom Instructions
- **Cursor**: 프로젝트 루트의 `.cursorrules` 파일에 저장
- **ChatGPT (Plus)**: Custom GPT의 instructions 영역
- 그 외: 매 대화 시작 시 한 번 붙여 넣어도 무방

### 3. 작업 프롬프트 사용

같은 [`prompt-pack.md`](./prompt-pack.md) 의 1~7번 작업 프롬프트를 필요할 때 골라서 복붙합니다.

- 1번: 파일 업로드 기능 추가
- 2번: 업로드한 파일 목록 보기
- 3번: 파일 삭제
- 4번: 이미지 미리보기
- 5번: 공개/비공개 전환
- 6번: 비용·사용량 위젯
- 7번: 에러 핸들링 강화

---

## 한계점

이 보조 경로는 다음과 같은 차이가 있습니다.

- AI가 매 대화에서 컨텍스트를 다시 받아야 함 → Skill 대비 토큰 소모 ↑
- 자동 활성화가 없음 → 사용자가 직접 시스템 프롬프트를 등록해야 함
- 보조 references(`object-storage.md`, `conventions.md`)를 AI가 자동으로 참조하지 않음 → 필요 시 본 레포 링크를 함께 던져야 함

가능하면 **Claude Code + Skill 경로**를 권장하지만, 다른 에디터에 익숙한 사용자도 이 경로로 충분히 같은 결과를 얻을 수 있습니다.
