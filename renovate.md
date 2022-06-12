### ブランチを指定して解析

```bash
RENOVATE_BASE_BRANCHES=${BRANCH} renovate --dry-run --token=${GITHUB_TOKEN} --log-file=renovate.log ${GITHUB_ORG}/${REPOSITORY}
```
