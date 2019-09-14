# Envoy
## Overview
モダンな大規模サービスオリエンテッドアーキテクチャシステムのためのL7プロキシ、サービスバス

サービスメッシュにおけるサイドカープロキシのデータプレーンとして利用される。（コントロールプレーンはIstioなど。AWSではAppMeshがその役割を担当）

> ![image](https://user-images.githubusercontent.com/1415655/63643166-a21bb700-c705-11e9-877e-633b455be527.png)
> https://philcalcado.com/2017/08/03/pattern_service_mesh.html


# Archtecture overview
## Introduction
### Terminology
#### Host
NW通信可能な論理的エンティティを表す。例えば仮想サーバーやモバイルフォンなども。

#### Downstream
Host >> Envoyへの通信の流れを表す。Envoyに対し送信したリクエストや返却されるレスポンスなど。

#### Upstream
Envoy >> Hostへの通信の流れを表す。EnvoyからHostに送信したリクエストや返却されるレスポンスなど。

#### Listener
Envoyが公開するDownstream HostsがEnvoyに接続するためのリスナー（port、unix socketなど）

#### Cluster
Upstream Hostsの論理的なグループを表す。
EnvoyはService DescoveryによりClusterのメンバーを検出し、Active Health Checkによりメンバーのヘルスを判断する。また負荷分散ポリシーによりリクエストをメンバーに分散する。

#### Mesh
Envoy Meshは、多くのサービスから構成される分散システム間のメッセージ送信回路を形成するEnvoyプロキシのグループを表す。

#### Runtime Configuration
Envoyは再起動したりすることなく設定を変更することができる。

### Threading model
シングルプロセス・マルチスレッドモデル。コネクションはリスナーにacceptされた後、単一のワーカースレッドにより全て処理される。基本ノンブロッキングになるように記述されており、ほとんどのワークロードでは、ワーカースレッド数はハードウェアスレッドと同じ数にすることが推奨されている（ハイパースレッディングのコア数）

## Listeners
Envoyは一つのプロセスで複数のListenerを持つことができる（単一のHostに単一のEnvoyプロセスが推奨されるアーキテクチャ）。現在はTCPリスナーのみサポートされている。それぞれのListenerはのNetwork filters（L3/L4）、およびListener filtersを定義することができる。

Listenerは「listener discovery service (LDS)」により動的設定することができる。

## HTTP
HTTPはモダンなサービス志向アプリケーションにおいて非常に重要なコンポーネントであり、Envoyでも多数の機能をサポートしている。

### HTTP Protocols
1.1, WebSocket, 2をサポート

### HTTP header sanitizing
セキュリティ的な理由により、EnvoyプロキシはいくつかのHTTPヘッダをサニタイズしたり、追加や転送といった処理を行う。

#### e.g. x-request-id
Envoyプロキシは全ての外部オリジンのリクエストに対し、`x-request-id` ヘッダを付与する。オリジナルのリクエストに値が存在していた場合、サニタイズされる。

#### e.g. x-forwarded-for
`use_remote_addressオプション`がtrueで、`skip_xff_appendオプション`がfalseの場合、EnvoyプロキシはXFFヘッダに最も近いクライアントのリモートIPを付与する。

### Route table configuration
VirtualHostベースのルーティングや、パスのリライト、重み付けによる分散など様々なリクエストのルーティングを行うことができる。
静的な設定の他に、「Route discovery service （RDS）」による動的設定が可能。

### Internal redirects
Envoyプロキシは、Upstreamサーバーから返却された３０２レスポンスをキャプチャし、ルートにマッチする新しいアップストリームに送信し、レスポンスを元のリクエストへのレスポンスとすることができる。

### Timeouts
様々なタイムアウトを管理

## Upstream clusters
### Cluster manager
Cluster managerはUpstreamのClusterの構成情報を管理する。
構成情報の登録は静的な設定の他に、「Cluster discovery service (CDS)」による動的な設定が可能。

### Cluster warming
Clusterの初期化時、warmingが行われる。
サービス検出され、ヘルスチェックが設定されている場合は、それに合格するまで、Envoy Proxyからは存在しないものとして扱われる。（HTTPルートを設定している場合、404や503になる）。Cluster更新時は、新しいClusterがウォームアップされるまで、古いClusterにトラフィックが送信される。ウォームアップが完了すると、トラフィックが切断しないよう、アトミックにClusterが交換される。

#### TODO

