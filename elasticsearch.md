### API
##### すべてのindexからaliasを列挙
```
curl http://localhost:9200/_aliases/
```

##### index を列挙

```
curl localhost:9200/_cat/indices?v
```


##### index削除
```
curl -XDELETE http://localhost:9200/{indexname}
```


### concepts
- NRT (Near Realtime
  - ほぼリアルタイム。検索可能になるまでにわずかな待ち時間（通常は1秒）はある。
  
### 要素
- cluster
  - A cluster is a collection of one or more nodes (servers)
  - すべてのNodeに渡るインデキシングと検索機能を提供する
  - cluster名重要。nodeは名前をもとにクラスタに参加する
  - デフォルトでは名前はelasticserchになる。
- node
  - A node is a single server that is part of your cluster
  - nodeはデフォルトではUUIDの名前を持つ。個別に設定することもできる
  - 現在のネットワークに別のnodeが存在しない場合、一つのnodeを起動すると`elasticseach`の名前で単一nodeを持つclusterが作成される
- index 
  - 同様の性質をもつdocumentのコレクション
- document
  - 情報の基本単位
- alias
  - indexの別名
  - 同じaliasを複数のindexにつけることもできるし、１つのindexに複数のaliasをつけることもできます
  - indexでなくaliasに対し検索を行うことで、実際の検索対象をelasticsearch側でコントロールすることができる
  - 別名以外にも任意の絞り込み条件であらかじめフィルタリングしておけるフィルター機能と、ドキュメントの物理的な配置 シャード をコントロールする為のルーティング機能が提供される
- sharding
  - indexの作成時に、シンプルに必要な数のシャードを定義できる。
  - 各シャードはそれ自体が、クラスタ内のノード上でホストされるフル機能の独立した「index」
- replication
  - 1つ以上のindexのシャードをいわゆるレプリカシャード（略してレプリカ）にコピーできる
  - シャード/ノードに障害が発生した場合の高可用性が提供されます。この理由で、レプリカシャードをコピー元のオリジナル/プライマリと同じノードに割り当てないように注意することが大切です。
  - 検索はすべてのレプリカで並列に実行できる
- sharding & replication 要約
  - indexは複数のシャードに分割されます。
  - indexはゼロ個以上のレプリカを作成できます。
  - レプリカが作成されると、各indexはプライマリシャード（レプリカを作成したオリジナルのシャード）とレプリカシャード（プライマリシャードのコピー）を持ちます。 
  - シャードとレプリカの数は、indexの作成時にindexごとに定義されます。
  - indexが作成された後、レプリカの数はいつでも動的に変更できますが、シャードの数は後になってからは変更できません。
  - デフォルトでは、各indexは5つのプライマリシャードと1つのレプリカを割り当てられます。
  - これは、クラスタに少なくとも2つのノードがある場合、indexは、indexにつき合計10個のシャードのうち、5つのプライマリシャードと5つのレプリカシャード（完全なレプリカ1つ）を持つということです。
  

### 制限
- 一つのシャードは、一つのLuceneindex。シャードにおけるドキュメント数の上限は2,147,483,519（= Integer.MAX_VALUE - 128）
- cat-shards APIを使用してシャードのサイズを監視できます
  
