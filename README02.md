# セクション3: アプリケーション開発編

## 6. Action Textに必要なマイグクレーションを実行する

+ `$ docker-compose exec web ./bin/rails -T`でタスクが閲覧できる<br>

+ `$ bundle exec rake action_text:install`を実行<br>

+ `$ bunle exec rake db:migrate`を実行<br>

## 7. image_processingをインストール

+ `Gemfile`の `gem 'image_processing', '~> 1.2'`をコメントアウトして`$ bundle install`する<br>

## 8. 投稿(post)をscaffoldする

+ `$ rails g scaffold post title:string`を実行<br>

+ `$ rails db:migrate`を実行

## 9 PostとRichTextを関連付ける

+ `app/models/post.rb`を編集<br>

```
class Post < ApplicationRecord
  has_rich_text :content
end
```

+ `$ rails console -s`を実行<br>

```
irb(main):001:0> post = Post.create!(content: "<p>HEllo, ActionText!</p>")
   (0.5ms)  BEGIN
   (0.4ms)  SAVEPOINT active_record_1
  Post Create (3.9ms)  INSERT INTO "posts" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", "2021-12-31 10:44:59.579512"], ["updated_at", "2021-12-31 10:44:59.579512"]]
  ActionText::RichText Create (16.6ms)  INSERT INTO "action_text_rich_texts" ("name", "body", "record_type", "record_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5, $6) RETURNING "id"  [["name", "content"], ["body", "<p>HEllo, ActionText!</p>"], ["record_type", "Post"], ["record_id", 2], ["created_at", "2021-12-31 10:45:00.234866"], ["updated_at", "2021-12-31 10:45:00.234866"]]
  ActiveStorage::Attachment Load (11.4ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = $1 AND "active_storage_attachments"."record_type" = $2 AND "active_storage_attachments"."name" = $3  [["record_id", 1], ["record_type", "ActionText::RichText"], ["name", "embeds"]]
  Post Update (6.1ms)  UPDATE "posts" SET "updated_at" = $1 WHERE "posts"."id" = $2  [["updated_at", "2021-12-31 10:45:00.257438"], ["id", 2]]
   (1.6ms)  RELEASE SAVEPOINT active_record_1
=> #<Post id: 2, title: nil, created_at: "2021-12-31 10:44:59", updated_at: "2021-12-31 10:45:00">
```

```
irb(main):002:0> post
=> #<Post id: 2, title: nil, created_at: "2021-12-31 10:44:59", updated_at: "2021-12-31 10:45:00">
```

```
irb(main):003:0> post.content
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 10.8ms | Allocations: 438)
=> #<ActionText::RichText id: 1, name: "content", body: #<ActionText::Content "<div class=\"trix-conte...">, record_type: "Post", record_id: 2, created_at: "2021-12-31 10:45:00", updated_at: "2021-12-31 10:45:00">
```

```
irb(main):004:0> post.content.to_s
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 1.8ms | Allocations: 204)
=> "<div class=\"trix-content\">\n  <p>HEllo, ActionText!</p>\n</div>\n"
```

```
irb(main):005:0> post.content.to_plain_text
=> "HEllo, ActionText!"
```

```
irb(main):006:0> post.content.class
=> ActionText::RichText(id: integer, name: string, body: text, record_type: string, record_id: integer, created_at: datetime, updated_at: datetime)
```

```
irb(main):007:0> post.content.class.table_name
=> "action_text_rich_texts"
```
