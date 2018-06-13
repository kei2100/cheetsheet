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
