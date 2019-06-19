### file
##### lsofみたいな

sysinternals suiteに入っているhandle.exeを利用

```powershell
handle.exe ファイルの名前など検索キーワード
handle.exe -p プロセスID
```

##### 共有ドライブの割り当て

```
net use p: ¥¥192.168.1.111¥share <password> /user:<username>
```
