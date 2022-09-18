### ranking of collection size

```ruby
Rails.application.eager_load!
db = MyModel.collection.database
Mongoid.models.
  reject { |m| m.embedded? }.
  map { |m|
    begin
        stats = db.command(collStats: m.collection.name).documents.first
        { total: stats[:size] + stats[:totalIndexSize], name: m.collection.name, collection: stats[:size], index: stats[:totalIndexSize]}
    rescue Mongo::Error::OperationFailure => e
      next if e.message.include?('not found')
      raise e
    end
  }.
  reject { |st| st.blank? }.
  sort{ |st1, st2| st2[:total] <=>  st1[:total] }.
  map { |st| st.merge! total: (st[:total] / 1024 / 1024 /1024).round(2).to_s + 'GB' }.
  first(20).
  each { |st| p st }; nil
```

### ドキュメントフィールドサイズの適当な計算

```
d, attrs = ExampleModel.first, ExampleModel.attribute_names
attrs.reduce(0) { |sum, a| puts "#{a}: #{d.send(a).to_s.bytesize}"; sum + d.send(a).to_s.bytesize }
```
