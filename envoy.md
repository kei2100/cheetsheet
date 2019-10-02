# Envoy
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

#### Cluster warming
Clusterの初期化時、warmingが行われる。
サービス検出され、ヘルスチェックが設定されている場合は、それに合格するまで、Envoy Proxyからは存在しないものとして扱われる。（HTTPルートを設定している場合、404や503になる）。Cluster更新時は、新しいClusterがウォームアップされるまで、古いClusterにトラフィックが送信される。ウォームアップが完了すると、トラフィックが切断しないよう、アトミックにClusterが交換される。

### Service discovery

### Health checking

### Connection pooling

### Load Balancing

### Outlier detection
異常値の検出と削除

### Circuit breaking

### Upstream network filters

## Observability
### Statistics

### Access logging

### Tracing

## Security
### TLS

### JWT Authentication

### External Authorization

### Role Based Access Control

## Operations & Configuration
### Dynamic configuration

### Initialization

### Draining

### Runtime configuration

### Hot restart

### Overload manager
オーバーロードマネージャーは、大量のクライアント接続やリクエスト過多によるシステムリソースの枯渇から（メモリ、CPU、ファイル記述子など）、Envoyサーバーを保護するための拡張可能なコンポーネント。

## Other features
### Global rate limiting

### Scripting

### IP Transparency
#### HTTP Headers

#### Proxy Protocol

#### Original Source Listener Filter

#### Original Source HTTP Filter

## Other protocols
### gRPC

### MongoDB

### DynamoDB

### Redis

## Advanced
### Sharing data between filters

# Deployment types
## Service to service
![image](https://user-images.githubusercontent.com/1415655/66081964-1d457800-e5a4-11e9-9de1-78a6039f777a.png)

## Front proxy
![image](https://user-images.githubusercontent.com/1415655/66081988-26cee000-e5a4-11e9-88c1-72b2b63fc064.png)

## Double proxy
![image](https://user-images.githubusercontent.com/1415655/66082018-32220b80-e5a4-11e9-9045-775099e60890.png)


# Getting Started

```bash
$ docker pull envoyproxy/envoy-alpine
$ docker run --rm -d -p 10000:10000 envoyproxy/envoy-alpine:latest
```

で、デフォルトだと*.google.comへのプロキシとして起動する。
このときの設定は以下の通り

```yml
$ docker exec -it 246f9a537172 cat /etc/envoy/envoy.yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 127.0.0.1
      port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite: www.google.com
                  cluster: service_google
          http_filters:
          - name: envoy.router
  clusters:
  - name: service_google
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    # Comment out the following line to test on v6 networks
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_google
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.google.com
                port_value: 443
    tls_context:
      sni: www.google.com
```

adminを以下に変更して、localhost:9901にアクセスすると管理画面を参照できる

```yml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      #address: 127.0.0.1
      address: 0.0.0.0
      port_value: 9901
```

```bash
$ docker run --rm -p 10000:10000 -p 9901:9901 -v $(pwd):/etc/envoy envoyproxy/envoy-alpine:latest
```
