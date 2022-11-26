### text
##### 標準入力同士でdiff
```bash
bash$ diff <(cat txt1) <(cat txt2) 
```

##### json diff
```bash
$ diff <(jq --sort-keys . 1_1.json) <(jq --sort-keys . 1_2.json)
```

##### Decimal to Hex
```bash
$ printf '%x\n' 909915735
363c3657
```

##### カラムを左揃え右揃え

```
$ echo -e "a 1234\n\
xyz 999999" | awk '{printf "%-15s%10s\n",$1,$2}'

a                    1234
xyz                999999
```

##### 出力を縦に
```
$ echo foobar | grep -o .
f
o
o
b
a
r
```

##### 各行80文字までで切り捨て
```
$ cat foo.txt | cut -c 1,80
```


##### 文字コード変換
iconv
```
cat foo.csv | iconv -f SHIFT_JIS -t UTF-8
```

nkf
```
cat foo.csv | nkf -S
```

##### awkでX区切りの最後の要素を出力
```例えばfindしてファイル名のみ出力
$ find ./proto/* -name '*.proto' | awk -F '/' '{print $NF}'
```

##### 文字列をデリミタで分割

```
# , delimiter
$ for i in $(echo "foo,bar,buz" | tr "," "\n"); do echo $i; done
```

##### 指定行に挿入

```bash
# mac
の場合は gsed (gnu-sed) で 
$ cat text 
a
b
c

$ cat text | gsed '2ifoo'
a
foo
b
c

# 末尾に挿入
$ cat text | gsed '$ a bar'
a
b
c
bar
```

##### base64

```bash
$ echo '<base64 str>' | base6 -d
```

##### 文字列置換

```
# perl
## 
perl -pe 's/str/rts/' text

## ファイル置換
perl -pi -e 's/str/rts/' file

## 各行ではなくファイル全体にマッチ. e.g. 末尾空行を一つ入れる
perl -0 -pi -e 's/(\r\n|[\n\r\u2028\u2029\u0085])*\z/\n/' 
```

### io
##### echo改行表示
```bash
$ echo -e '\na'

a
-- 
$ echo $'\n'a

a
```

##### echo改行無し
```bash
echo -n 'aaa'
```

##### cat ヒアドキュメント リダイレクト
```
cat << EOF > doc.txt
  abc
  def
EOF
```

### process
##### プロセスの環境変数
```bash
$ cat /proc/<pid>/environ | strings
```

##### psで表示項目選択
```bash
$ ps -o user,group,pid,ppid,c,start_time,tty,time,comm
```

##### プロセス起動時刻
```bash
$ ps -eo lstart,pid,args
```

- lstart	コマンドが実行された日時
- pid	プロセスID
- args	コマンドと引数

### network
##### 疎通確認
```bash
# TCP 
$ nc <host> <port>

# UDP
$ nc -u <host> <port>

# 送信元の指定
$ nc -s <from_ip> -p <from_port> <host> <port>

# TCP listen
$ nc -l <port>

# UDP listen
$ nc -lu <port>
```

#### 帯域モニタリングツール
iptraf
nethogs

#### DNS 
##### txtレコードの確認
```bash
$ nslookup -q=txt <domain_name>
```
##### dig
```bash
# ANSWERセクションだけ表示
dig hoo.com +noall +answer

```

#### tcpdump
```bash
# ASCII表示でループバックアドレス:８０への通信をダンプ
$ tcpdump -A -i lo0 port 80 

```

#### ss
##### TCP・UDPソケットの確認
```bash
# netstat -tan (--tcp --all --numeric)
$ ss -tan

# netstat -uan (--udp --all --numeric)
$ ss -uan
```

### navigate
##### 前のディレクトリに戻る
```bash
$ cd -
```

##### mkdirした先に移動
```bash
$ mkdir foo && cd $_
# $_ means the last argument to the previous command.
```

##### scriptのディレクトリ取得
```bash
CURDIR=$(cd $(dirname $0) && pwd)
```

##### symlinkの実体に移動
```
$ cd -P <link>
```

### filesystem
##### symlink保持して zip
```
$ cp -R

$ zip dst.zip -yr src
```

##### ファイルのバイト数
標準入力でパスを渡すときれいにバイト数だけ取れる
```
wc -c < /path/to/file
```

##### 指定した時間よりあとに更新されたファイル

```
$ find ./* -newermt '20221124 10:11:37' 
```

### OS
##### version
```
# os
$ uname -a

# ubuntu
$ cat /etc/os-release

# centos
cat /etc/redhat-release

# amazon linux
cat /etc/system-release

# alipine
cat /etc/alpine-release 
```

##### プロセスが読み込んでいる環境変数表示
```
strings /proc/[PID]/environ
```

##### envdirでaccess denied

```
# sudo -u postgres envdir /etc/wal-e.d/env whoami
envdir: fatal: unable to switch to current directory: access denied

メッセージ通りだがカレントディレクトリにpostgresユーザーでアクセスできない場合、このようなエラーになるので注意
```

### pkg
##### alipineのedgeブランチ追加
```bash
echo http://dl-cdn.alpinelinux.org/alpine/edge/main/ >> /etc/apk/repositories
echo http://dl-cdn.alpinelinux.org/alpine/edge/community/ >> /etc/apk/repositories
```

##### インストール済みpkg一覧
```bash
# alpine
apk info
```

##### pkgがインストールするファイル一覧
```bash
# alipne
apk info -L <package>
```

### tls
##### 証明書の情報表示
```bash
openssl x509 -in etc/letsencrypt/live/hoge.hoge/cert.pem -text -noout

# from web site
openssl s_client -connect example.com:443 < /dev/null 2> /dev/null | openssl x509 -noout -text

# 証明書チェーンの内容表示ワンライナー
openssl s_client -servername server.example.com -connect server.example.com:443 -showcerts </dev/null >c.pem
openssl crl2pkcs7 -nocrl -certfile c.pem | openssl pkcs7 -print_certs -text -noout
```

##### CSR pemの情報表示
```bash
openssl req -in ./csr.pem -text -noout
```

##### CRL pemの情報表示
```bash
openssl crl -in ./crl.pem -text -noout 
```

##### RSA key pemの情報表示
```bash
openssl rsa -in ./key.pem -text -noout 
```

##### DER binary 2 PEM text
```
openssl x509 -in cert.der -out cert.pem -inform DER -outform PEM
```

##### p12(pfx) to pem
```
openssl pkcs12 -in client.pfx -clcerts -nokeys -out cert.pem  # クライアント証明書
openssl pkcs12 -in client.pfx -nocerts -nodes -out key.pem
```

##### private key から public key を生成

```
# PKCS#8 形式の ec (elliptic curve) 秘密鍵ファイルから公開鍵を生成
openssl ec -in private.p8 -pubout -out public.p8
```

##### サイトのTLSバージョン確認

```
openssl s_client -connect api.stripe.com:443 -tls1 < /dev/null
openssl s_client -connect api.stripe.com:443 -tls1_1 < /dev/null
openssl s_client -connect api.stripe.com:443 -tls1_2 < /dev/null
```

### daemon
#### 起動
```bash
# systemd
sudo systemctl start docker

# other
sudo service docker start
```

#### 自動起動設定
```bash
# systemd
sudo systemctl enable docker

# other
sudo chkconfig docker on
```

### time
##### YYYYMMDDHHMISS
```bash
date '+%Y%m%d%H%M%S'
```

##### iso8601
```bash
$ date --iso-8601
2018-06-12

$ date --iso-8601=hours
2018-06-12T15+09:00

$ date --iso-8601=minutes
2018-06-12T15:56+09:00

$ date --iso-8601=seconds
2018-06-12T15:56:32+09:00
```

### tty
##### terminalの入力がおかしくなったとき



```bash
# 入力文字が表示されない
# returnで改行されない
$ stty sane

# cursor が表示されない
$ reset
```

##### terminal へのペーストで「0~hoge~1」のように 0~ ~1 で対象が囲まれてしまうとき

```
printf "\e[?2004l"
```

##### exec で標準出力をファイルにリダイレクト

```
$ exec > output

# コマンドの結果は output に出力される
$ ls -l

# 出力をターミナルに戻す
$ exec > /dev/tty
```


### http
##### JSONをパイプしてPOST

```bash
$ echo '
{
  "insurance_card": {
    "id": "2a7798f7-37c6-40e0-86e6-c63db5859002"
  }
}' | curl -XPOST https://example.com/insurance_cards -d @-
```
