## 23 バリデーションを実装する - 添付ファイルのファイル数

```
[53] pry(main)> post = Post.find(3)
  Post Load (6.6ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
=> #<Post:0x00005578abb99a00 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sat, 08 Jan 2022 07:32:27 UTC +00:00>
[54] pry(main)>
[55] pry(main)> post
=> #<Post:0x00005578abb99a00 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sat, 08 Jan 2022 07:32:27 UTC +00:00>
[56] pry(main)>
```

```
[58] pry(main)> post.content // postにはcontentが紐づいている
  ActionText::RichText Load (9.1ms)  SELECT "action_text_rich_texts".* FROM "action_text_rich_texts" WHERE "action_text_rich_texts"."record_id" = $1 AND "action_text_rich_texts"."record_type" = $2 AND "action_text_rich_texts"."name" = $3 LIMIT $4  [["record_id", 3], ["record_type", "Post"], ["name", "content"], ["LIMIT", 1]]
=> #<ActionText::RichText:0x00005578adaf4490  ActiveStorage::Blob Load (9.0ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 125.1ms | Allocations: 2601)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 297.3ms | Allocations: 12185)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 3,
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sat, 08 Jan 2022 07:32:27 UTC +00:00>
[59] pry(main)>
```

```
 show-source ActionText::RichText

 From: /app/vendor/bundle/gems/actiontext-6.0.4.4/app/models/action_text/rich_text.rb:4
Class name: ActionText::RichText
Number of monkeypatches: 4. Use the `-a` option to display all available monkeypatches
Number of lines: 19

class RichText < ActiveRecord::Base
  self.table_name = "action_text_rich_texts"

  serialize :body, ActionText::Content
  delegate :to_s, :nil?, to: :body

  belongs_to :record, polymorphic: true, touch: true
  has_many_attached :embeds

  before_save do
    self.embeds = body.attachables.grep(ActiveStorage::Blob).uniq if body.present?
  end

  def to_plain_text
    body&.to_plain_text.to_s
  end

  delegate :blank?, :empty?, :present?, to: :to_plain_text
end
[62] pry(main)>
```

```
[64] pry(main)> post.content.body
=>   ActiveStorage::Blob Load (1.0ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 1.7ms | Allocations: 517)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 18.9ms | Allocations: 2326)
#<ActionText::Content "<div class=\"trix-conte...">
[65] pry(main)>
```

```
[67] pry(main)> post.content.body.attachables
  ActiveStorage::Blob Load (1.4ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
=> [#<ActiveStorage::Blob:0x00005578ac10a778
  id: 1,
  key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
  filename: "001.jpg",
  content_type: "image/jpeg",
  metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
  byte_size: 3629588,
  checksum: "13znRaP8NLezkHEv5lVLfA==",
  created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>]
[68] pry(main)>
```

```
[69] pry(main)> post.content.body.attachables.grep(ActiveStorage::Blob)
=> [#<ActiveStorage::Blob:0x00005578ac10a778
  id: 1,
  key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
  filename: "001.jpg",
  content_type: "image/jpeg",
  metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
  byte_size: 3629588,
  checksum: "13znRaP8NLezkHEv5lVLfA==",
  created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>]
[70] pry(main)>
```

```
[72] pry(main)> post.content.body.attachables.grep(ActiveStorage::Blob).count
=> 1
[73] pry(main)>
```

- `app/models/post.rb`を編集<br>

```
class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length
  validate :validate_content_attachment_byte_size
  validate :validate_content_attachments_count

  MAX_CONTENT_LENGTH = 50
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 4
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE
  MAX_CONTENT_ATTACHMENTS_COUNT = 4

  private

  def validate_content_length
    length = content.to_plain_text.length

    if length > MAX_CONTENT_LENGTH
      errors.add(
        :content,
        :too_long,
        max_content_length:  MAX_CONTENT_LENGTH,
        length: length
      )
    end
  end

  def validate_content_attachment_byte_size
    content.body.attachables.grep(ActiveStorage::Blob).each do |attachable|
      if attachable.byte_size > MAX_CONTENT_ATTACHMENT_BYTE_SIZE
        errors.add(
          :base,
          :content_attachment_byte_size_is_too_big,
          max_content_attachment_mega_byte_size: MEGA_BYTES,
          bytes: attachable.byte_size,
          max_bytes: MAX_CONTENT_ATTACHMENT_BYTE_SIZE
        )
      end
    end
  end

  def validate_content_attachments_count
    if content.body.attachables.grep(ActiveStorage::Blob).count > MAX_CONTENT_ATTACHMENTS_COUNT
      errors.add(
        :content,
        :attachments_count_is_too_big,
        max_content_attachments_count: MAX_CONTENT_ATTACHMENTS_COUNT
      )
    end
  end
end
```

- `config/locales/ja.yml`を編集<br>

```
ja:
  activerecord:
    errors:
      models:
        post:
          attributes:
            base:
              content_attachment_byte_size_is_too_big: 添付ファイルは%{max_content_attachment_mega_byte_size}MB以下にしてください。[%{bytes}/%{max_bytes}]
            title:
              blank: が空です。
              too_long: が%{count}文字を超えています。
            content:
              too_long: が%{max_content_length}文字を超えています。[%{length}/%{max_content_length}]
              attachments_count_is_too_big: の添付画像の数が%{max_content_attachments_count}個を超えています。
    attributes:
      post:
        title: タイトル
        content: コンテンツ
```
