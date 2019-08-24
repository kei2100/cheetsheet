# Envoy
## Overview
モダンな大規模サービスオリエンテッドアーキテクチャシステムのためのL7プロキシ、サービスバス

サービスメッシュにおけるサイドカープロキシのデータプレーンとして利用される。（コントロールプレーンはIstioなど。AWSではAppMeshがその役割を担当）

> ![image](https://user-images.githubusercontent.com/1415655/63643166-a21bb700-c705-11e9-877e-633b455be527.png)
> https://philcalcado.com/2017/08/03/pattern_service_mesh.html


## Terminology
### Host
NW通信可能な論理的エンティティを表す。例えば仮想サーバーやモバイルフォンなども。

### Downstream
Host >> Envoyへの通信の流れを表す。Envoyに対し送信したリクエストや返却されるレスポンスなど。

### Upstream
Envoy >> Hostへの通信の流れを表す。EnvoyからHostに送信したリクエストや返却されるレスポンスなど。

### Listener
Envoyが公開するDownstream HostsがEnvoyに接続するためのリスナー（port、unix socketなど）

### Cluster
Upstream Hostsの論理的なグループを表す。
EnvoyはService DescoveryによりClusterのメンバーを検出し、Active Health Checkによりメンバーのヘルスを判断する。また負荷分散ポリシーによりリクエストをメンバーに分散する。

### Mesh
Envoy Meshは、多くのサービスから構成される分散システム間のメッセージ送信回路を形成するEnvoyプロキシのグループを表す。

### Runtime Configuration
Envoyは再起動したりすることなく設定を変更することができる。
