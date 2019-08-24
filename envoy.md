# Envoy
## Overview
モダンな大規模サービスオリエンテッドアーキテクチャシステムのためのL7プロキシ、サービスバス

サービスメッシュにおけるサイドカープロキシのデータプレーンとして利用される。（コントロールプレーンはIstioなど。AWSではAppMeshがその役割を担当）

> ![image](https://user-images.githubusercontent.com/1415655/63643166-a21bb700-c705-11e9-877e-633b455be527.png)
> https://philcalcado.com/2017/08/03/pattern_service_mesh.html


### 構成例

## Terminology
### Host
NW通信可能な論理的エンティティを表す。例えば仮想サーバーやモバイルフォンなども。

### Downstream
Host >> Envoyへの通信の流れを表す。Envoyに対する

