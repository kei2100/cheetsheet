### docker ps で COMMANDも表示したい

` docker ps --no-trunc`

### win mount & powershell
`docker run -v c:/gopath:c:/gopath -it golang:1.12.4 powershell`

### docker-compose で volume が更新されない
一度 volume を削除する。
```
docker volume ls
docker volume rm <volume name>
```
