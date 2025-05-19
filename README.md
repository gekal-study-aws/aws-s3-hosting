# AWS S3のホスティング

## 手順

### バケット作成

```bash
aws s3api create-bucket \
    --bucket gekal-aws-s3-hosting-demo \
    --region ap-northeast-1 \
    --create-bucket-configuration LocationConstraint=ap-northeast-1
```

### バケットポリシーの設定

```bash
aws s3api delete-public-access-block \
    --bucket gekal-aws-s3-hosting-demo
```

```bash
aws s3api put-bucket-policy \
    --bucket gekal-aws-s3-hosting-demo \
    --policy '{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "PublicReadGetObject",
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::gekal-aws-s3-hosting-demo/*"
        }
      ]
    }'
```

### 静的ウェブホスティングの有効化

```bash
aws s3 website s3://gekal-aws-s3-hosting-demo/ \
    --index-document index.html \
    --error-document error.html
```

### ファイルのアップロード

```bash
aws s3 sync dist s3://gekal-aws-s3-hosting-demo/ --delete
# set cache control
aws s3 sync dist s3://gekal-aws-s3-hosting-demo/ --delete --cache-control "no-cache, no-store"
```

### 注意⚠️

1. 既存のファイルを上書きでアップロードする時に、`cache-control`の指定が反映されない。
2. AWS Consoleからも直接編集できない。
   1. オブジェクトメタデータを更新するには、改善された`[コピー]`アクションを使用して`[設定を指定]`を選択します。

## 動作確認

<https://gekal-aws-s3-hosting-demo.s3.ap-northeast-1.amazonaws.com/index.html>

### JSONのレスポンスの確認

```shell
# without cache header
$ curl --head https://gekal-aws-s3-hosting-demo.s3.ap-northeast-1.amazonaws.com/api/persons.json
HTTP/1.1 200 OK
x-amz-id-2: 9kjJpof88AVtqa4jsHIA462sTbxicfZTB3WO29kpo6Wi2c9Uj3VE90ydwKLFRMD+bBz9LCWgxf8=
x-amz-request-id: MWC3MYFWRFXZYKV4
Date: Mon, 19 May 2025 21:29:49 GMT
Last-Modified: Mon, 19 May 2025 21:28:25 GMT
ETag: "94a74a5558699aa42faaa69e482291d6"
x-amz-server-side-encryption: AES256
Accept-Ranges: bytes
Content-Type: application/json
Content-Length: 929
Server: AmazonS3

# with cache header
$ curl --head https://gekal-aws-s3-hosting-demo.s3.ap-northeast-1.amazonaws.com/api/persons.json
HTTP/1.1 200 OK
x-amz-id-2: m6zqRKnRTLTQODG3gpYfTKMDe/6N0Ul5MSJc83JLFPzXPY5zVlTkIikjIyqy0iD7dn2wdnjOPsQ=
x-amz-request-id: SBGEJJF1N07TQZMA
Date: Mon, 19 May 2025 21:33:12 GMT
Last-Modified: Mon, 19 May 2025 21:32:55 GMT
ETag: "94a74a5558699aa42faaa69e482291d6"
x-amz-server-side-encryption: AES256
Cache-Control: no-cache, no-store
Accept-Ranges: bytes
Content-Type: application/json
Content-Length: 929
Server: AmazonS3
```

## 環境クリア

### バケット削除

```bash
aws s3 rb s3://gekal-aws-s3-hosting-demo --force
```

## 参照

1. [Amazon S3 を使用して静的ウェブサイトをホスティングする](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/WebsiteHosting.html)
