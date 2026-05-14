# NCP Server(VM) 패턴

본 문서는 ncloud-cli로 NCP VPC Server를 만들고 운영하는 표준 패턴이다.
새 계정은 기본적으로 VPC 환경을 사용하므로, 모든 명령은 `ncloud vserver ...` 형태를 따른다.

## 1. 콘솔 사전 준비 (CLI 자동 생성 금지)

서버를 만들기 전에 네 가지 사전 자원이 필요하다. **모두 NCP 콘솔에서 사용자가 직접 만들도록 안내한다.** CLI로 자동 생성하지 않는다 (실수 시 영향 큼).

| 자원 | 콘솔 메뉴 | 비고 |
| --- | --- | --- |
| VPC | Services > Networking > VPC | IPv4 CIDR 예: `10.0.0.0/16` |
| Subnet | VPC > Subnet 관리 | Public Subnet, KR-1 또는 KR-2 |
| ACG (보안그룹) | VPC > ACG | 기본은 SSH(22)만 허용으로 시작 |
| Login Key | Server > Login Key 관리 | `.pem` 파일 다운로드 즉시 보관 |

각 자원의 번호(`vpcNo`, `subnetNo`, `accessControlGroupNo`, `loginKeyName`)를 메모해 둔다. 서버 생성 명령의 필수 인자다.

## 2. 사용 가능한 이미지·사양 조회

서버 생성 전, 어떤 OS와 사양이 가능한지 확인한다.

```bash
# OS 이미지 목록
ncloud vserver getServerImageList --regionCode KR

# 인스턴스 사양(상품) 목록
ncloud vserver getServerProductList --regionCode KR --serverImageProductCode <위에서 고른 이미지 코드>
```

**기본 추천**: Ubuntu 22.04 LTS, vCPU 2 / RAM 4GB 등급 (마이크로 또는 standard 가장 저렴한 것).

## 3. 서버 생성

```bash
ncloud vserver createServerInstances \
  --regionCode KR \
  --vpcNo <VPC 번호> \
  --subnetNo <Subnet 번호> \
  --serverImageProductCode <이미지 코드> \
  --serverProductCode <사양 코드> \
  --loginKeyName "<프로젝트명>-key" \
  --networkInterfaceList "networkInterfaceOrder='0',subnetNo='<Subnet 번호>',accessControlGroupNoList='<ACG 번호>'" \
  --serverName "<프로젝트명>-api"
```

응답 JSON의 `serverInstanceNo` 가 이 서버의 식별자다. 메모.

## 4. 상태 확인 / 공인 IP 할당

생성 직후 서버는 `INIT` → `SETUP` → `RUN` 으로 상태가 진행된다. 보통 1~3분 소요.

```bash
ncloud vserver getServerInstanceList --regionCode KR --serverInstanceNoList "<serverInstanceNo>"
```

`serverInstanceStatusName: running` 가 되면 공인 IP를 할당한다.

```bash
ncloud vserver createPublicIpInstance \
  --regionCode KR \
  --serverInstanceNo <serverInstanceNo>
```

응답에서 `publicIp` 값을 사용자에게 전달.

## 5. SSH 접속

Login Key의 `.pem` 파일이 있는 디렉토리에서:

```bash
chmod 600 <프로젝트명>-key.pem
ssh -i <프로젝트명>-key.pem ubuntu@<공인 IP>
```

ACG에 22번 포트(SSH)가 인바운드로 열려 있어야 한다. 콘솔 사전 준비 단계에서 이미 열려 있을 것.

## 6. 백엔드 배포 패턴 (Node.js 예시)

```bash
# VM 내부에서
sudo apt-get update && sudo apt-get install -y nodejs npm git

git clone https://github.com/<your-name>/<your-repo>.git ~/app
cd ~/app
npm ci

# 프로세스 매니저
sudo npm install -g pm2
pm2 start npm --name api -- start
pm2 startup systemd
pm2 save
```

Python(FastAPI) 예시:

```bash
sudo apt-get install -y python3 python3-venv
python3 -m venv ~/.venv && source ~/.venv/bin/activate
pip install -r requirements.txt
sudo apt-get install -y nginx           # 리버스 프록시용(선택)
uvicorn main:app --host 0.0.0.0 --port 8000 &
```

운영 안정성을 위해서는 `systemd` 또는 `pm2`로 묶는 것을 권장한다.

## 7. ACG 인바운드 규칙 추가 (앱 포트 열기)

기본 ACG는 SSH(22)만 열려 있다. 백엔드가 사용하는 포트(예: 3000, 80, 443)를 추가로 열어야 외부에서 접근 가능하다.

```bash
ncloud vserver addNetworkInterfaceAccessControlGroup \
  --regionCode KR \
  --networkInterfaceNo <NIC 번호> \
  --accessControlGroupNoList <ACG 번호>
```

ACG 자체에 규칙을 추가하려면 콘솔의 **ACG > 규칙 설정** 화면에서 인바운드 규칙을 추가하도록 안내하는 것이 안전하다 (CLI로 ACG 규칙 추가는 인자 실수 시 영향 크다).

규칙 예시:
- 프로토콜: TCP, 포트: 3000 (Node), 소스: `0.0.0.0/0` (테스트용. 운영에서는 특정 CIDR로 좁힐 것)
- 프로토콜: TCP, 포트: 80, 소스: `0.0.0.0/0`
- 프로토콜: TCP, 포트: 443, 소스: `0.0.0.0/0`

## 8. 비용 정리(중요)

실습 후에는 다음을 정리한다.

```bash
# 서버 정지
ncloud vserver stopServerInstances \
  --regionCode KR \
  --serverInstanceNoList "<serverInstanceNo>"

# 서버 반납 (과금 종료)
ncloud vserver terminateServerInstances \
  --regionCode KR \
  --serverInstanceNoList "<serverInstanceNo>"

# 공인 IP 반납 (할당된 채로 방치하면 과금됨)
ncloud vserver deletePublicIpInstance \
  --regionCode KR \
  --publicIpInstanceNo "<publicIpInstanceNo>"
```

VPC·Subnet·ACG·Login Key는 무료이므로 그대로 두어도 무방하다.

## 9. 흔한 에러

| 증상 | 가능한 원인 | 해결 |
| --- | --- | --- |
| 서버 생성 실패: `Authorization` 에러 | `~/.ncloud/configure` 자격증명 오타 / Sub Account 권한 부족 | `references/credentials.md` 다시 확인 |
| 생성은 되는데 SSH 안 됨 | ACG 22번 포트 미허용 / `.pem` 권한 문제 | `chmod 600`, ACG 규칙 확인 |
| 공인 IP 할당 실패: `quota` | 신규 계정 IP 쿼터 한도 도달 | 콘솔에서 미사용 IP 반납 또는 한도 상향 요청 |
| 앱 포트(예: 3000)로 외부 접근 안 됨 | ACG 인바운드 미허용 | 7번 항목 참조 |
| 비용이 계속 발생함 | 서버 정지만 하고 반납 안 함 / 공인 IP 미반납 | 8번 항목 참조. `terminate` + IP `delete` 모두 필요 |
