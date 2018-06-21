
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
