### go バイナリの go get

```bash
$ go install golang.org/dl/go1.17@latest

# goX.XX コマンドがインストールされる

$ go1.17 download
Downloaded   0.0% (    16384 / 135938157 bytes) ...
Downloaded   2.8% (  3801072 / 135938157 bytes) ...
Downloaded   6.1% (  8339408 / 135938157 bytes) ...
Downloaded  96.3% (130923552 / 135938157 bytes) ...
Downloaded  99.7% (135511040 / 135938157 bytes) ...
Downloaded 100.0% (135938157 / 135938157 bytes)
Success. You may now run 'go1.17'

# goX.XX で X.XX バージョンの go コマンドを実行できる

$ go1.17 version
go version go1.17 darwin/amd64
```

### docker image pull ってバイナリビルド

CGO_ENABLED=1 しやすい
```
docker run --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp golang:1.20.5 env GOOS=linux GOARCH=amd64 CGO_ENABLED=1 go build -ldflags="-s -w" -o bin/app/linux/amd64 ./app
```

### tool get & install

```
$ go get -tool github.com/foo/bar@latest
$ go install tool
```

### ディレクトリ配下のインタフェースをカンマ区切りで

```
$ go doc -all -u . | grep "^type .* interface" | awk '{print $2}' | paste -sd "," -
```
