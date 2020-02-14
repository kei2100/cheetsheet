## terraform state
### mv 

terraform.state.from のリソースをモジュール化して terraform.state に移動（マージ可）

```bash
terraform state list -state=terraform.tfstate.from | xargs -I{} -n 1 \
  terraform state mv -state=terraform.tfstate.from -state-out=terraform.tfstate {} module.vpc.{}
```
