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
aws ecr get-login --region=ap-northeast-1 --no-include-email | sh
```

## AWS Account
#### アカウント番号取得
```bash
aws sts get-caller-identity --output=text --query='Account'
```
