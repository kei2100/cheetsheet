### ブランチを指定して解析

```bash
RENOVATE_BASE_BRANCHES=${BRANCH} renovate --dry-run --token=${GITHUB_TOKEN} --log-file=renovate.log ${GITHUB_ORG}/${REPOSITORY}
```

### Custom Manager Support using Regex

https://docs.renovatebot.com/modules/manager/regex/

```json
{
  "regexManagers": [
    {
      "fileMatch": ["^Dockerfile$"],
      "matchStrings": [
        "datasource=(?<datasource>.*?) depName=(?<depName>.*?)( versioning=(?<versioning>.*?))?\\sENV .*?_VERSION=(?<currentValue>.*)\\s"
      ],
      "versioningTemplate": "{{#if versioning}}{{{versioning}}}{{else}}semver{{/if}}"
    },
    {
      "fileMatch": ["^Dockerfile$"],
      "matchStrings": [
        "ARG IMAGE=(?<depName>.*?):(?<currentValue>.*?)@(?<currentDigest>sha256:[a-f0-9]+)s"
      ],
      "datasourceTemplate": "docker"
    }
  ]
}
```
