# NCP Object Storage 패턴

NCP Object Storage는 AWS S3 호환이다. 대부분의 작업은 `aws cli` 또는 AWS SDK를 그대로 쓸 수 있다.
엔드포인트는 `https://kr.object.ncloudstorage.com`, 리전은 `kr-standard` 고정.

## 1. 버킷 라이프사이클

### 생성

```bash
BUCKET="<프로젝트명>-<용도>-$(openssl rand -hex 2)"
aws --profile ncp --endpoint-url "$NCP_ENDPOINT" s3 mb "s3://$BUCKET" --region kr-standard
```

### 목록 / 삭제

```bash
aws --profile ncp --endpoint-url "$NCP_ENDPOINT" s3 ls

# 비우기 → 삭제 (순서 주의)
aws --profile ncp --endpoint-url "$NCP_ENDPOINT" s3 rm "s3://$BUCKET" --recursive
aws --profile ncp --endpoint-url "$NCP_ENDPOINT" s3 rb "s3://$BUCKET"
```

## 2. 파일 업로드 / 다운로드

```bash
# 업로드
aws --profile ncp --endpoint-url "$NCP_ENDPOINT" s3 cp ./local.png "s3://$BUCKET/<key>"

# 다운로드
aws --profile ncp --endpoint-url "$NCP_ENDPOINT" s3 cp "s3://$BUCKET/<key>" ./downloaded.png
```

## 3. Node.js / TypeScript (서버 라우트에서 사용)

```ts
// app/api/upload/route.ts (Next.js 예시)
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";

const s3 = new S3Client({
  region: "kr-standard",
  endpoint: "https://kr.object.ncloudstorage.com",
  credentials: {
    accessKeyId: process.env.NCP_ACCESS_KEY!,
    secretAccessKey: process.env.NCP_SECRET_KEY!,
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

브라우저에서 직접 호출하지 말 것. 반드시 서버 라우트 경유.

## 4. Python (boto3)

```python
import boto3, os

s3 = boto3.client(
    "s3",
    endpoint_url="https://kr.object.ncloudstorage.com",
    region_name="kr-standard",
    aws_access_key_id=os.environ["NCP_ACCESS_KEY"],
    aws_secret_access_key=os.environ["NCP_SECRET_KEY"],
)

# 업로드
s3.upload_file("local.png", os.environ["NCP_BUCKET"], "uploads/local.png")

# 객체 목록
resp = s3.list_objects_v2(Bucket=os.environ["NCP_BUCKET"])
for obj in resp.get("Contents", []):
    print(obj["Key"], obj["Size"])
```

## 5. 공개 / 비공개

기본은 비공개. 임시 공개가 필요하면 **Presigned URL**을 발급한다.

```python
url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": bucket, "Key": key},
    ExpiresIn=300,   # 5분
)
```

## 6. CORS

브라우저에서 직접 호출해야 하는 경우(예: presigned URL로 PUT), 버킷에 CORS 정책을 설정한다.

```bash
cat > cors.json <<EOF
{
  "CORSRules": [{
    "AllowedOrigins": ["https://<your-domain>"],
    "AllowedMethods": ["GET", "PUT", "POST"],
    "AllowedHeaders": ["*"],
    "MaxAgeSeconds": 3000
  }]
}
EOF

aws --profile ncp --endpoint-url "$NCP_ENDPOINT" \
  s3api put-bucket-cors --bucket "$BUCKET" --cors-configuration file://cors.json
```

## 7. 흔한 에러 처리

| 에러 | 흔한 원인 | 해결 |
| --- | --- | --- |
| `403 AccessDenied` | 키 권한 부족, 또는 ACL 설정 문제 | Sub Account 권한 확인, ACL 검사 |
| `404 NoSuchBucket` | 리전 불일치, 또는 버킷 이름 오타 | `--region kr-standard` 명시 확인 |
| `SignatureDoesNotMatch` | Secret Key 오타, 또는 시계 차이 | `aws configure --profile ncp` 재확인 |
| CORS preflight 실패 | 버킷 CORS 미설정 | 6번 항목 참조 |
