## AZ
### ゾーン ID の取得

```bash
$ aws ec2 describe-availability-zones --region ap-northeast-1
{
    "AvailabilityZones": [
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "ap-northeast-1",
            "ZoneName": "ap-northeast-1a",
            "ZoneId": "apne1-az4",
            "GroupName": "ap-northeast-1",
            "NetworkBorderGroup": "ap-northeast-1",
            "ZoneType": "availability-zone"
        },
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "ap-northeast-1",
            "ZoneName": "ap-northeast-1c",
            "ZoneId": "apne1-az1",
            "GroupName": "ap-northeast-1",
            "NetworkBorderGroup": "ap-northeast-1",
            "ZoneType": "availability-zone"
        },
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "ap-northeast-1",
            "ZoneName": "ap-northeast-1d",
            "ZoneId": "apne1-az2",
            "GroupName": "ap-northeast-1",
            "NetworkBorderGroup": "ap-northeast-1",
            "ZoneType": "availability-zone"
        }
    ]
}
```

## S3
#### ページングして最後のオブジェクトを表示
using: awscli, jq

```bash
BUCKET='foo'
PREFIX='bar'

MAX_ITEMS="1000"
STARTING_TOKEN=""
OBJECTS=""

while :
do
  OBJECTS=$(aws s3api list-objects --max-items ${MAX_ITEMS} --bucket ${BUCKET} --prefix ${PREFIX} ${STARTING_TOKEN})
  if [ -z "${OBJECTS}" ]; then
    echo "s3api list-objects returns empty response."
    echo "path: ${BUCKET}/${PREFIX} starting-token: ${STARTING_TOKEN}"
    exit 1
  fi

  NEXT_TOKEN=$(echo "${OBJECTS}" | jq -r '.NextToken')
  if [ "${NEXT_TOKEN}" != "null" ]; then
    STARTING_TOKEN="--starting-token ${NEXT_TOKEN}"
    continue
  fi

  break
done

echo "${OBJECTS}" | jq -r '.Contents | .[-1].Key'
```

## ECR
#### ログインワンライナー
using: awscli
```bash
# deprecated
aws ecr get-login --region=ap-northeast-1 --no-include-email | sh

# recommended
aws ecr get-login-password | docker login --username AWS --password-stdin https://${AWS_ACCOUNT}.dkr.ecr.ap-northeast-1.amazonaws.com
```

## AWS Account
#### アカウント番号取得
```bash
aws sts get-caller-identity --output=text --query='Account'
```

## LocalStack
#### Endpoint 指定

awscli で LocalStack にアクセスするときは Endpoint 指定が必要な場合がある

```bash
# localhost:4566 で localstack sqs がリスンしているとして
aws --endpoint-url=http://localhost:4566 sqs get-queue-attributes --queue-url=http://localhost:4566/queue/my-queue
```
