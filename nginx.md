### upstreamのログを出力

```
log_format upstreamlog '$remote_addr - $remote_user - [$time_local] "$request" '
                       'to: $upstream_addr: $upstream_status '
                       'upstream_response_time $upstream_response_time request_time $request_time';

access_log  logs/access.upstream.log  upstreamlog; 
```

http://nginx.org/en/docs/http/ngx_http_upstream_module.html
