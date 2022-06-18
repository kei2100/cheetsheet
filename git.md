## 個人的にignoreしたい

`.git/info/exclude`に書き込む

```
/.idea/
/.envrc
/*.iml
.DS_Store
```

## merge commitのrevert

```
git show <マージコミットのhash> する

commit 8ae52de41110e4a27c765f6b68f5a3c19b0dd199
Merge: 0e2695a47 2082dfba2

0e2695a47がマージコミット（親）ならば、「-m 1」
2082dfba2が親ならば「-m 2」として、

git revert 0e2695a47 -m 1
のようにする
```

## gitignoreで特定の拡張子だけ管理に含みたい
```
/example/*
!/example/*.go  # これは含む
```

## 特定のファイルだけ revert したい

```
git checkout {戻したい位置の commit hash} {ファイルパス}
```

## パーシャルクローンとシャロークローン

```bash
# ブロブレスクローン
## 到達可能なすべてのコミットとツリーをダウンロードするが、ブロブは必要に応じて取得する
$ git clone --filter=blob:none <url>

# ツリーレスクローン
## 到達可能なすべてのコミットをダウンロードするが、ツリーとブロブは必要に応じて取得する
$ git clone --filter=tree:0 <url>

# シャロークローン
## コミット履歴を切り捨ててクローンサイズを小さくする
$ git clone --depth=1 <url>
```
