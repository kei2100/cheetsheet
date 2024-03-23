# Ruby
## gem のインストールパス
```
$ gem which rails
```

## Benchmark

```ruby
Benchmark.bm do |x|
  x.report("benchmark1: ") { process1() }
  x.report("benchmark2: ") { process2() }
end
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

# RSpec
## ジョブエンキューの matcher

```ruby
expect { subject }.to have_enqueued_job.on_queue(queue_name).with { |job_class, args|
  expect(job_class).to eq(want_job_class)
  expect(args).to eq(want_args)
}
```
