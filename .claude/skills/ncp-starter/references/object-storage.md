# Object Storage 보조 패턴 (선택)

> 본 문서는 **Server 시나리오에 파일 저장이 추가로 필요할 때**만 참조한다.
> 본 스킬의 메인은 Server(VM)다. 단독으로 Object Storage만 다루는 시나리오는 후속 편에서 별도 다룬다.

## 1. 도구 선택 원칙

| 작업 종류 | 권장 도구 |
| --- | --- |
| 버킷 생성·삭제·메타데이터 조회 | `ncloud objectstorageservice` 명령 |
| 콘솔에서 직접 클릭하는 파일 업로드/다운로드 | NCP 콘솔 웹 UI |
| 애플리케이션 코드에서의 파일 입출력 | `boto3` (Python), `@aws-sdk/client-s3` (JS/TS) |
| CLI에서 단발성 파일 업로드 | 콘솔 사용 권장. 또는 SDK 기반 짧은 스크립트 |

NCP Object Storage는 S3 호환 API를 제공하므로, **애플리케이션 코드 레벨**에서는 AWS SDK 사용이 NCP가 공식 문서에서 권장하는 방식이다. CLI 레벨에서는 `aws cli` 가 아닌 `ncloud-cli` 와 콘솔을 사용한다.

## 2. 버킷 라이프사이클 (ncloud-cli)

### 생성

```bash
BUCKET="<프로젝트명>-<용도>-$(openssl rand -hex 2)"
ncloud objectstorageservice createBucket \
  --regionCode KR \
  --bucketName "$BUCKET"
```

### 목록

```bash
ncloud objectstorageservice getBucketList --regionCode KR
```

### 삭제

```bash
# 비우는 작업은 콘솔 또는 SDK 스크립트로 진행한 뒤
ncloud objectstorageservice deleteBucket \
  --regionCode KR \
  --bucketName "$BUCKET"
```

> `ncloud objectstorageservice` 는 버킷 단위의 관리에 적합하다. 파일 단위 조작이 필요하면 콘솔 또는 SDK를 사용한다.

## 3. 코드에서 파일 입출력 (SDK)

### Node.js / TypeScript (서버 라우트)

```ts
// app/api/upload/route.ts (Next.js 예시)
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";

const s3 = new S3Client({
  region: "kr-standard",
  endpoint: "https://kr.object.ncloudstorage.com",
  credentials: {
    accessKeyId: process.env.NCLOUD_ACCESS_KEY!,
    secretAccessKey: process.env.NCLOUD_SECRET_KEY!,
  },
});

export async function POST(req: Request) {
  const form = await req.formData();
  const file = form.get("file") as File;
  const buf = Buffer.from(await file.arrayBuffer());
  const key = `${Date.now()}-${file.name}`;

  await s3.send(new PutObjectCommand({
    Bucket: process.env.NCP_BUCKET!,
    Key: key,
    Body: buf,
    ContentType: file.type,
  }));

  return Response.json({ ok: true, key });
}
```

브라우저에서 직접 호출하지 말 것. 반드시 서버 라우트(또는 백엔드)에서 처리.

### Python (boto3)

```python
import boto3, os

s3 = boto3.client(
    "s3",
    endpoint_url="https://kr.object.ncloudstorage.com",
    region_name="kr-standard",
    aws_access_key_id=os.environ["NCLOUD_ACCESS_KEY"],
    aws_secret_access_key=os.environ["NCLOUD_SECRET_KEY"],
)

s3.upload_file("local.png", os.environ["NCP_BUCKET"], "uploads/local.png")

resp = s3.list_objects_v2(Bucket=os.environ["NCP_BUCKET"])
for obj in resp.get("Contents", []):
    print(obj["Key"], obj["Size"])
```

## 4. 공개 / 비공개

기본은 비공개. 임시 공개가 필요하면 SDK로 **Presigned URL** 을 발급한다.

```python
url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": bucket, "Key": key},
    ExpiresIn=300,   # 5분
)
```

## 5. 흔한 에러

| 에러 | 흔한 원인 | 해결 |
| --- | --- | --- |
| 코드에서 `403 AccessDenied` | 키 권한 부족 / ACL 문제 | Sub Account 권한 확인, ACL 검사 |
| `404 NoSuchBucket` | 리전 불일치 / 버킷 이름 오타 | SDK 호출 시 `region_name="kr-standard"` 확인 |
| `SignatureDoesNotMatch` | Secret Key 오타 / 시계 차이 | `.ncloud/configure` 재확인, 시간 동기화 |

## 6. 안내 멘트 (사용자에게)

사용자가 Object Storage 사용을 처음 시작할 때, 다음 사실을 한 줄로 알린다.

> "NCP Object Storage는 S3 호환이라, 애플리케이션 코드에서는 AWS SDK(boto3, @aws-sdk/client-s3)를 그대로 사용합니다. 이는 NCP가 공식 권장하는 방식이며, AWS 계정과는 무관합니다. 버킷 관리·CLI 작업은 ncloud-cli와 NCP 콘솔로 진행합니다."

이 한 문장으로 사용자의 "왜 AWS인가?" 의문을 해소한다.
