### text
##### 標準入力同士でdiff
```bash
bash$ diff <(cat txt1) <(cat txt2) 
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



