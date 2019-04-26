### net
##### pfでパケットフィルター

```bash
pfctl -f /etc/pf.conf  # ルールの反映
pfctl -nf /etc/pf.conf  # syntax チェック
pfctl -d  # pf を無効化する
pfctl -e  # pf を有効化する
pfctl -sr  # 反映されているルールの確認
```

pf.conf例
```
# ループバックのxxxxxポートから、yyyyyポートへのinパケットをドロップする
block in on lo0 proto tcp from self port xxxxx to self port yyyyy
```

[書式](https://man.openbsd.org/pf.conf)


### Finder
##### 隠しファイル表示
`cmd + shift + .`

### OS
##### 登録URLスキーム一覧

```
/System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister -dump | egrep "bindings.+:" | sort
```
