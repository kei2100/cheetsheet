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

### navigate
##### 前のディレクトリに戻る
```bash
$ cd -
```
