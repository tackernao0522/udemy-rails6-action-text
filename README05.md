## 23 バリデーションを実装する - 添付ファイルのサイズ

```
root@7820c712b618:/app# bundle exec rails c -s
Loading development environment in sandbox (Rails 6.0.4.4)
Any modifications you make will be rolled back on exit
[1] pry(main)> post = Post.find(3)
   (0.5ms)  BEGIN
  Post Load (10.8ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
=> #<Post:0x00005578ae29a708 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[2] pry(main)>
```

```
[4] pry(main)> post
=> #<Post:0x00005578ae29a708 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[5] pry(main)>
```

```
[8] pry(main)> content = post.content
=> #<ActionText::RichText:0x00005578ac964450  ActiveStorage::Blob Load (0.9ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 2.0ms | Allocations: 517)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 13.5ms | Allocations: 2606)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 3,
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[9] pry(main)>
```

```
[11] pry(main)> content
=> #<ActionText::RichText:0x00005578ac964450  ActiveStorage::Blob Load (1.9ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 2.6ms | Allocations: 517)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 41.0ms | Allocations: 2604)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 3,
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[12] pry(main)>
```

```
[14] pry(main)> show-source ActionText::RichText

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
[15] pry(main)>
```

```
[16] pry(main)> content.body.attachables
  ActiveStorage::Blob Load (1.6ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
=> [#<ActiveStorage::Blob:0x00005578ae3ba3b8
  id: 1,
  key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
  filename: "001.jpg",
  content_type: "image/jpeg",
  metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
  byte_size: 3629588,
  checksum: "13znRaP8NLezkHEv5lVLfA==",
  created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>]
[18] pry(main)>
```

```
[21] pry(main)> content.body.attachables.grep(ActiveStorage::Blob)
=> [#<ActiveStorage::Blob:0x00005578ae3ba3b8
  id: 1,
  key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
  filename: "001.jpg",
  content_type: "image/jpeg",
  metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
  byte_size: 3629588,
  checksum: "13znRaP8NLezkHEv5lVLfA==",
  created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>]
[22] pry(main)>
```

```
[24] pry(main)> attachable = content.body.attachables.grep(ActiveStorage::Blob).first
=> #<ActiveStorage::Blob:0x00005578ae3ba3b8
 id: 1,
 key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
 filename: "001.jpg",
 content_type: "image/jpeg",
 metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
 byte_size: 3629588,
 checksum: "13znRaP8NLezkHEv5lVLfA==",
 created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>
[25] pry(main)>
```

```
[28] pry(main)> attachable
=> #<ActiveStorage::Blob:0x00005578ae3ba3b8
 id: 1,
 key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
 filename: "001.jpg",
 content_type: "image/jpeg",
 metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
 byte_size: 3629588,
 checksum: "13znRaP8NLezkHEv5lVLfA==",
 created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>
[29] pry(main)>
```

```
[32] pry(main)> attachable.byte_size
=> 3629588
[33] pry(main)>
```

```
[36] pry(main)> attachable.byte_size / (1024 * 1_000.0)
=> 3.54451953125 // 約3.5Mの大きさ
[37] pry(main)>
```

```
edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length
  validate :validate_content_attachment_byte_size

  MAX_CONTENT_LENGTH = 50
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 3
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE

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
end
```

```
[42] pry(main)> reload!
Reloading...
=> true
[43] pry(main)>
```

```
[44] pry(main)> Post::MAX_CONTENT_ATTACHMENT_BYTE_SIZE
=> 3072000
[45] pry(main)>
```

```
[45] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length
  validate :validate_content_attachment_byte_size

  MAX_CONTENT_LENGTH = 50
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 3
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE

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
          :content_attachment_byte_size_is_too_big
        )
      end
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
              content_attachment_byte_size_is_too_big: 添付ファイルは3MB以下にしてください。[/]
            title:
              blank: が空です。
              too_long: が%{count}文字を超えています。
            content:
              too_long: が%{max_content_length}文字を超えています。[%{length}/%{max_content_length}]
    attributes:
      post:
        title: タイトル
        content: コンテンツ
```

```
[48] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length
  validate :validate_content_attachment_byte_size

  MAX_CONTENT_LENGTH = 50
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 3
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE

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
        )
      end
    end
  end
end
```

```
[48] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length
  validate :validate_content_attachment_byte_size

  MAX_CONTENT_LENGTH = 50
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 3
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE

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
        )
      end
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
              content_attachment_byte_size_is_too_big: 添付ファイルは%{max_content_attachment_mega_byte_size}MB以下にしてください。[%{bytes}/]
            title:
              blank: が空です。
              too_long: が%{count}文字を超えています。
            content:
              too_long: が%{max_content_length}文字を超えています。[%{length}/%{max_content_length}]
    attributes:
      post:
        title: タイトル
        content: コンテンツ
```

```
[50] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length
  validate :validate_content_attachment_byte_size

  MAX_CONTENT_LENGTH = 50
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 3
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE

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
    attributes:
      post:
        title: タイトル
        content: コンテンツ
```

```
[50] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length
  validate :validate_content_attachment_byte_size

  MAX_CONTENT_LENGTH = 50
  ONE_KILOBYTE = 1024
  MEGA_BYTES = 4 // 変更して通るようになる
  MAX_CONTENT_ATTACHMENT_BYTE_SIZE = MEGA_BYTES * 1_000 * ONE_KILOBYTE

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
end
```
