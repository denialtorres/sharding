## Test. 

# Validate that there is not elements in shard one database

```ruby
ActiveRecord::Base.connected_to(role: :reading, shard: :shard_one) do
  Person.first
end
```
