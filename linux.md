### text
##### 標準入力同士でdiff
```bash
bash$ diff <(cat txt1) <(cat txt2) 
```

##### Decimal to Hex
```bash
$ printf '%x\n' 909915735
363c3657
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

#### DNS 
##### txtレコードの確認
```bash
$ nslookup -q=txt <domain_name>
```

##### tcpdump
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
##### symlink保持して
```
$ cp -R

$ zip dst.zip -yr src
```

##### ファイルのバイト数
標準入力でパスを渡すときれいにバイト数だけ取れる
```
wc -c < /path/to/file
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

### tls
##### 証明書pemの情報表示
```bash
openssl x509 -in etc/letsencrypt/live/hoge.hoge/cert.pem -text -noout
```

##### CSR pemの情報表示
```bash
openssl req -in ./csr.pem -text -noout
```

##### CRL pemの情報表示
```bash
 openssl crl -in ./crl.pem -text -noout 
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

- 入力文字が表示されない
- returnで改行されない


```
stty sane
```
