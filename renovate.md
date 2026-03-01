### ローカルの設定ファイルで dry run

```bash
LOG_LEVEL=debug RENOVATE_CONFIG_FILE=renovate.json5 renovate --dry-run --platform=local > tmp/renovate.log
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

### 設定ファイルのバリデーション

```bash
$ npx --package renovate -- renovate-config-validator renovate.json5
```
