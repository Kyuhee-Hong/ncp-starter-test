# Prompt Pack — 바이브코더용 (Server 중심)

> AI 에디터(Claude Desktop · Cursor · ChatGPT 등)에 그대로 복붙해서 쓰는 프롬프트 모음입니다.
> 본 팩을 활용하면 AI가 매번 같은 것을 추론하지 않아 **토큰을 아낄 수 있고**, 결과의 일관성도 올라갑니다.

---

## 사용법

1. **0. 시스템 프롬프트**를 자신의 정보로 채워 한 번만 붙입니다.
   - Claude Desktop: **Projects > 새 프로젝트 > Custom Instructions**
   - Cursor: 프로젝트 루트의 `.cursorrules` 파일
   - ChatGPT(Plus): Custom GPT의 instructions 영역
2. 이후 작업이 필요할 때마다 **1~N의 작업 프롬프트**를 골라 붙입니다.

---

## 0. 시스템 프롬프트 (한 번만 설정)

```
당신은 내가 NAVER Cloud Platform(NCP)에 백엔드 서버를 만들고 운영하는 것을 도와주는 어시스턴트입니다.

[지켜야 할 규칙]
1. 모든 NCP 자원 조작은 NCP 공식 CLI인 `ncloud-cli`로 수행한다.
   aws cli 는 NCP 자원에 사용하지 않는다.
2. 자격증명은 환경 변수(NCLOUD_ACCESS_KEY, NCLOUD_SECRET_KEY)에서 읽는다.
   코드나 명령에 키 값을 직접 박지 않는다.
3. 리소스 생성·삭제 전 사용자에게 한 번 더 확인을 받는다.
4. 네트워크 자원(VPC, Subnet, ACG, Login Key)은 CLI로 자동 생성하지 않고
   NCP 콘솔에서 사용자가 직접 만들도록 안내한다.
5. 한 번에 하나의 단계만 진행한다. 욕심내서 여러 단계를 묶지 않는다.
6. ncloud-cli 응답 JSON을 그대로 노출하지 말고, 핵심 필드만 표로 요약한다.

[내 환경]
- 프로젝트명: <my-project>
- 사용할 리전: KR
- 프론트엔드 프레임워크: <React / Next.js / Vue / 정적 HTML 중 하나>
- 백엔드 후보: <Node Express / Python FastAPI / 기타>

[참고할 명세]
- NCP CLI 명령은 `ncloud vserver <command> --regionCode KR` 형태.
- 서버 생성: ncloud vserver createServerInstances
- 서버 목록: ncloud vserver getServerInstanceList
- 공인 IP 할당: ncloud vserver createPublicIpInstance
- 서버 반납: ncloud vserver terminateServerInstances
- 사용자 환경 변수: NCLOUD_ACCESS_KEY, NCLOUD_SECRET_KEY, NCLOUD_API_URL

[작명 컨벤션]
- 서버: <프로젝트명>-<용도> (예: mysite-api)
- ACG: <프로젝트명>-acg-<용도>
- Login Key: <프로젝트명>-key
```

---

## 1. 작업: 처음으로 서버 한 대 띄우기

```
NCP에 백엔드 서버 한 대를 처음으로 띄우고 싶다.
다음 정보를 차례로 안내해 줘.

1. 사전에 콘솔에서 만들어야 할 자원 4가지(VPC, Subnet, ACG, Login Key)의 클릭 절차.
   - VPC CIDR, ACG 기본 인바운드 규칙(SSH 22번만)에 대한 추천 값을 함께 알려 줘.
2. ncloud vserver getServerImageList 와 getServerProductList 로
   Ubuntu 22.04 LTS / 마이크로(가장 저렴) 사양의 코드를 어떻게 찾아내는지.
3. ncloud vserver createServerInstances 명령의 최종 한 줄 예시.
   - 서버 이름은 <my-project>-api 로 한다.

한 단계씩 끝낼 때마다 내가 "다음"이라고 말하면 진행해 줘.
```

---

## 2. 작업: 공인 IP 할당 + SSH 접속

```
방금 만든 서버에 공인 IP를 할당하고 SSH로 접속하고 싶다.

1. ncloud vserver createPublicIpInstance 명령 예시.
2. Login Key(.pem) 파일에 권한 적용하는 chmod 명령.
3. ssh -i ... ubuntu@<공인IP> 명령 예시.
4. 접속 실패 시 점검 순서(ACG 22번 포트, pem 권한, 공인 IP 상태).
```

---

## 3. 작업: 백엔드 코드 배포 (Node Express)

```
서버에 Node Express 백엔드를 배포하고 싶다.
- 코드는 내 GitHub 레포에 있다고 가정한다.
- 프로세스 매니저로 pm2 를 사용한다.
- 포트는 3000번을 사용한다.

다음을 안내해 줘.
1. VM 안에서 Node/npm/git 설치하는 명령.
2. 레포 클론 → 의존성 설치 → pm2 로 백그라운드 실행하는 명령.
3. 재부팅 후 자동 시작되도록 pm2 startup 설정.
4. ACG에 3000번 포트 인바운드 규칙을 추가하는 절차(콘솔에서 클릭으로).
5. 외부에서 curl http://<공인IP>:3000/ 으로 동작 확인하는 명령.

한 단계씩 끝나면 내가 "다음"이라고 말한다.
```

---

## 4. 작업: 프론트엔드에 백엔드 URL 연결

```
프론트엔드 코드(<프레임워크>)에서 방금 띄운 백엔드를 호출하고 싶다.

1. 환경 변수로 백엔드 URL을 받는 패턴(.env, .env.local 등).
2. fetch / axios 호출 예시 코드.
3. CORS 이슈가 났을 때 백엔드(Express)에 cors 미들웨어를 추가하는 한 줄 코드.
4. 운영 환경에서 백엔드 URL을 HTTPS 로 바꿔야 하는 이유와 다음에 무엇을 해야 하는지 한 줄 안내.
```

---

## 5. 작업: 비용 정리 (실습 끝낼 때)

```
방금 만든 서버와 공인 IP를 모두 반납하고 싶다.
실수로 다른 자원을 건드리지 않게 주의하면서, 다음 순서로 안내해 줘.

1. 어떤 자원을 반납해야 비용이 종료되는지 한 줄 요약(서버, 공인 IP).
2. 어떤 자원을 그대로 두어도 비용이 안 드는지(VPC, Subnet, ACG, Login Key).
3. ncloud vserver stopServerInstances → terminateServerInstances 순서의 명령.
4. ncloud vserver deletePublicIpInstance 명령.
5. NCP 콘솔의 Billing > 요금 현황에서 무엇을 확인해야 하는지.

각 명령 실행 직전에는 내가 "확인"이라고 말한 뒤 진행해 줘.
```

---

## 부록 A: 트러블슈팅 프롬프트

### A1. 서버 생성 시 Authorization 에러

```
ncloud vserver createServerInstances 실행 중 Authorization 에러가 났다.
가능한 원인 순서대로 점검 항목을 알려 줘.
하나씩 확인하도록 단계별로 안내해 줘.
```

### A2. SSH 접속 안 됨

```
ssh -i <project>-key.pem ubuntu@<공인IP> 가 응답이 없다(Connection timed out).
원인 가설을 가능성 높은 순서대로 1~4개 정리하고, 각각 어떻게 검증할 수 있는지 알려 줘.
ACG 인바운드 규칙 확인 절차를 가장 먼저 다뤄 줘.
```

### A3. 외부에서 앱 포트로 접근 안 됨

```
서버 안에서 curl localhost:3000 은 정상인데, 내 PC에서 curl http://<공인IP>:3000 이 응답이 없다.
원인 가설과 점검 순서를 정리해 줘. ACG 규칙과 0.0.0.0 바인딩 여부를 가장 먼저 다뤄 줘.
```

---

## 부록 B: (선택) Object Storage가 추가로 필요할 때

서버에 파일 업로드 기능을 붙이고 싶을 때만 사용합니다.

### B1. 버킷 만들기

```
NCP Object Storage에 새 버킷을 만들고 싶다.
버킷 관리는 ncloud objectstorageservice 명령을 사용한다.
다음을 안내해 줘.
1. ncloud objectstorageservice createBucket 명령 예시(버킷명: <project>-uploads-<random>).
2. ncloud objectstorageservice getBucketList 로 확인.
3. 콘솔에서 같은 버킷이 보이는지 안내.
```

### B2. 백엔드 코드에 업로드 라우트 추가

```
내 Express 백엔드에 파일 업로드 라우트를 추가하고 싶다.
- NCP Object Storage는 S3 호환이라 코드에서는 @aws-sdk/client-s3 를 사용한다(NCP 공식 권장 방식).
- 키 값은 .env(NCLOUD_ACCESS_KEY, NCLOUD_SECRET_KEY, NCP_BUCKET, NCP_OBJECT_STORAGE_ENDPOINT)에서 읽는다.
- 엔드포인트는 https://kr.object.ncloudstorage.com 로 고정.

코드와 .env.example 을 함께 보여 줘.
```

> NCP가 공식 문서에서 권장하는 방식이므로 "AWS SDK 사용"이 어색하게 들릴 수 있어도 문제 없습니다. AWS 계정과는 무관합니다.
