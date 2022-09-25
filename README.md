# Test. 

## Validate that there is not elements in shard one database

```ruby
ActiveRecord::Base.connected_to(role: :reading, shard: :shard_one) do
  Person.first
end
```

result:

```console
nil
```

## Now lets add elements to `shard_one`

```ruby
ActiveRecord::Base.connected_to(role: :writing, shard: :shard_one) do 
  Person.create!(name: "John Sharded")
  Person.create!(name: "Georgina Sharded")
end
```

result: 

```console
TRANSACTION (0.1ms)  begin transaction
  Person Create (5.1ms)  INSERT INTO "people" ("name", "created_at", "updated_at") VALUES (
?, ?, ?)  [["name", "John Sharded"], ["created_at", "2022-09-25 23:09:04.528450"], ["update
d_at", "2022-09-25 23:09:04.528450"]]
  TRANSACTION (1.2ms)  commit transaction
  TRANSACTION (0.1ms)  begin transaction
  Person Create (1.2ms)  INSERT INTO "people" ("name", "created_at", "updated_at") VALUES (
?, ?, ?)  [["name", "Georgina Sharded"], ["created_at", "2022-09-25 23:09:04.536631"], ["up
dated_at", "2022-09-25 23:09:04.536631"]]
  TRANSACTION (1.2ms)  commit transaction
 => #<Person id: 2, name: "Georgina Sharded", created_at: "2022-09-25 23:09:04.536631000 +0
000", updated_at: "2022-09-25 23:09:04.536631000 +0000">
```

## Now les read the values from `shard_one` replica

```ruby
ActiveRecord::Base.connected_to(role: :reading, shard: :shard_one) do 
  Person.count
  Person.pluck(:name)
end
```

result: 

```console
Person Count (0.9ms)  SELECT COUNT(*) FROM "people"
 => 2
 Person Pluck (0.2ms)  SELECT "people"."name" FROM "people"
 => ["John Sharded", "Georgina Sharded"]
```

## confirm that shards one data has not been written to the primary database

```ruby
ActiveRecord::Base.connected_to(role: :reading, shard: :default) do
  Person.count
  Person.pluck(:name)
end
```

result:
```console
Person Count (0.9ms)  SELECT COUNT(*) FROM "people"
 => 2
  Person Pluck (0.2ms)  SELECT "people"."name" FROM "people"
 => ["Frieda", "Bill", "Penelope"]
```
