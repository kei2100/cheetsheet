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

