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

### マルチプラットフォームでの sha 表示

```
$ docker buildx imagetools inspect alpine:3.19.0
Name:      docker.io/library/alpine:3.19.0
MediaType: application/vnd.docker.distribution.manifest.list.v2+json
Digest:    sha256:51b67269f354137895d43f3b3d810bfacd3945438e94dc5ac55fdac340352f48

Manifests:
  Name:      docker.io/library/alpine:3.19.0@sha256:13b7e62e8df80264dbb747995705a986aa530415763a6c58f84a3ca8af9a5bcd
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/amd64

  Name:      docker.io/library/alpine:3.19.0@sha256:45eeb55d6698849eb12a02d3e9a323e3d8e656882ef4ca542d1dda0274231e84
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm/v6

  Name:      docker.io/library/alpine:3.19.0@sha256:41f5f86616c51186dde18811bae696c689d6d492e1428f84fd74d42b43799c71
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm/v7

  Name:      docker.io/library/alpine:3.19.0@sha256:a70bcfbd89c9620d4085f6bc2a3e2eef32e8f3cdf5a90e35a1f95dcbd7f71548
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm64/v8

  Name:      docker.io/library/alpine:3.19.0@sha256:b31dd6ba7d28a1559be39a88c292a1a8948491b118dafd3e8139065afe55690a
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/386

  Name:      docker.io/library/alpine:3.19.0@sha256:5e9a2ed3363da2971d80ae49d9a084d1a6c20a8681ee34b958719a553be6bd3a
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/ppc64le

  Name:      docker.io/library/alpine:3.19.0@sha256:e247930c071ea57e24794a44bfbf3a20cbdf4c326471600dc2146356a2287e8a
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/s390x
```
