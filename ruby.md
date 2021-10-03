# Ruby
## gem のインストールパス
```
$ gem which rails
```

# Rails
## models
### validatorの内容確認

```ruby
ModelClass.validators
```

## routing
### consoleでxxx_pathの生成

```ruby
 app.oauth_authorization_path(client_id: client.uid, redirect_uri: client.redirect_uri, response_type: 'code', scope: client.scopes)
```

### routingの確認

rails routesだと毎回Rails環境を読み込んで遅いので、consoleでshow-routesするか、spring rake routesする
