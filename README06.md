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

## 25 ゴミデータを破壊する

+ `$ bundle exec rails c -s`を実行<br>

```
[1] pry(main)> ActiveStorage::Blob
=>    (0.5ms)  BEGIN
ActiveStorage::Blob(id: integer, key: string, filename: string, content_type: string, metadata: text, byte_size: integer, checksum: string, created_at: datetime)
[2] pry(main)>
```

```
[3] pry(main)> ActiveStorage::Blob.count
   (9.7ms)  SELECT COUNT(*) FROM "active_storage_blobs"
=> 9
[4] pry(main)>
```

+ ブラウザで新規登録で一枚画像をドラッグ&ドロップする(createはまだされていないのにサーバーログで永続化されている)<br>

```terminal:result
ActiveStorage::Blob Create (6.5ms)  INSERT INTO "active_storage_blobs" ("key", "filename", "content_type", "byte_size", "checksum", "created_at") VALUES ($1, $2, $3, $4, $5, $6) RETURNING "id"
```

```terminal:console
[4] pry(main)> ActiveStorage::Blob.count
   (1.3ms)  SELECT COUNT(*) FROM "active_storage_blobs"
=> 10 // createしていないのに10となっている
[5] pry(main)>
```

```
[6] pry(main)> ActiveStorage::Blob.pluck(:id)
   (10.1ms)  SELECT "active_storage_blobs"."id" FROM "active_storage_blobs"
=> [1, 2, 3, 4, 5, 7, 6, 8, 9, 10]
[7] pry(main)>
```

```
[8] pry(main)> blob_ids = ActiveStorage::Blob.pluck(:id)
   (1.3ms)  SELECT "active_storage_blobs"."id" FROM "active_storage_blobs"
=> [1, 2, 3, 4, 5, 7, 6, 8, 9, 10]
[9] pry(main)>
```

```
中間テーブル
[13] pry(main)> ActiveStorage::Attachment.pluck(:blob_id) // 使われている（表示）blob_id
   (15.4ms)  SELECT "active_storage_attachments"."blob_id" FROM "active_storage_attachments"
=> [1, 6, 7, 8]
[14] pry(main)>
```

```
[15] pry(main)> ActiveStorage::Attachment.pluck(:blob_id).uniq
   (1.4ms)  SELECT "active_storage_attachments"."blob_id" FROM "active_storage_attachments"
=> [1, 6, 7, 8]
[16] pry(main)>
```

```
[18] pry(main)> _blob_ids = ActiveStorage::Attachment.pluck(:blob_id).uniq
   (0.9ms)  SELECT "active_storage_attachments"."blob_id" FROM "active_storage_attachments"
=> [1, 6, 7, 8]
[19] pry(main)>
```

```
[22] pry(main)> blob_ids
=> [1, 2, 3, 4, 5, 7, 6, 8, 9, 10]
[23] pry(main)>
```

```
[24] pry(main)> _blob_ids
=> [1, 6, 7, 8]
[25] pry(main)>
```

```
[27] pry(main)> blob_ids - _blob_ids // 未使用のblob_id
=> [2, 3, 4, 5, 9, 10]
[28] pry(main)>
```

```
[28] pry(main)> unreferenced_blob_ids = blob_ids - _blob_ids
=> [2, 3, 4, 5, 9, 10]
[29] pry(main)>
```

```
[31] pry(main)> ActiveStorage::Blob
=> ActiveStorage::Blob(id: integer, key: string, filename: string, content_type: string, metadata: text, byte_size: integer, checksum: string, created_at: datetime)
[32] pry(main)>
```

```
[34] pry(main)> ActiveStorage::Blob.where(id: unreferenced_blob_ids) // 未使用のblobデータ
=>   ActiveStorage::Blob Load (1.6ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" IN ($1, $2, $3, $4, $5, $6)  [["id", 2], ["id", 3], ["id", 4], ["id", 5], ["id", 9], ["id", 10]]
[#<ActiveStorage::Blob:0x000055928932b010
  id: 2,
  key: "e7lt09ved273tzvefv40nw99k793",
  filename: "1489404.png",
  content_type: "image/png",
  metadata: {},
  byte_size: 231240,
  checksum: "EmO93/jV5xMd9eRY44PmjQ==",
  created_at: Sat, 08 Jan 2022 08:05:22 UTC +00:00>,
 #<ActiveStorage::Blob:0x00005592894405b8
  id: 3,
  key: "tmhbrcmpcp6pfn7bn6ks4pgeirog",
  filename: "2046435_s.jpg",
  content_type: "image/jpeg",
  metadata: {},
  byte_size: 108679,
  checksum: "emVltxXiCtu7dz1WrRYOGA==",
  created_at: Sat, 08 Jan 2022 08:05:22 UTC +00:00>,
 #<ActiveStorage::Blob:0x00005592894404f0
  id: 4,
  key: "s1k6y6jf89v5guyz8jhsufa301ou",
  filename: "1027174_s.jpg",
  content_type: "image/jpeg",
  metadata: {},
  byte_size: 200502,
  checksum: "HWwPhh6OhPIG6k5ZYl5jQw==",
  created_at: Sat, 08 Jan 2022 08:05:22 UTC +00:00>,
 #<ActiveStorage::Blob:0x0000559289440428
  id: 5,
  key: "mds1x6if6todgwlemw3byskpbx89",
  filename: "1317223_s.jpg",
  content_type: "image/jpeg",
  metadata: {},
  byte_size: 230229,
  checksum: "GTbA8IVcz1Z/TuvwqR3/Fw==",
  created_at: Sat, 08 Jan 2022 08:05:22 UTC +00:00>,
 #<ActiveStorage::Blob:0x0000559289440360
  id: 9,
  key: "c06egm9536s91hcte90vj5wvyujq",
  filename: "001.jpg",
  content_type: "image/jpeg",
  metadata: {},
  byte_size: 3629588,
  checksum: "13znRaP8NLezkHEv5lVLfA==",
  created_at: Sat, 08 Jan 2022 08:09:22 UTC +00:00>,
 #<ActiveStorage::Blob:0x0000559289440298
  id: 10,
  key: "4u4vdvwg9ncbz9qghy6op1kyhrj1",
  filename: "001.jpg",
  content_type: "image/jpeg",
  metadata: {},
  byte_size: 3629588,
  checksum: "13znRaP8NLezkHEv5lVLfA==",
  created_at: Mon, 10 Jan 2022 07:26:06 UTC +00:00>]
[35] pry(main)>
```

```
[44] pry(main)> unreferenced_blob_ids
=> [2, 3, 4, 5, 9, 10]
[45] pry(main)>
```

```
[46] pry(main)> unreferenced_blob_ids.count
=> 6 // 未使用データは6件
[47] pry(main)>
```

```
[47] pry(main)> ActiveStorage::Blob.where(id: unreferenced_blob_ids).delete_all
  ActiveStorage::Blob Destroy (24.8ms)  DELETE FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" IN ($1, $2, $3, $4, $5, $6)  [["id", 2], ["id", 3], ["id", 4], ["id", 5], ["id", 9], ["id", 10]]
=> 6 // 未使用データの6件が削除された
[48] pry(main)>
```

```
[48] pry(main)> ActiveStorage::Blob.where(id: unreferenced_blob_ids).delete_all
  ActiveStorage::Blob Destroy (3.2ms)  DELETE FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" IN ($1, $2, $3, $4, $5, $6)  [["id", 2], ["id", 3], ["id", 4], ["id", 5], ["id", 9], ["id", 10]]
=> 0 // 削除されたので0件になる
[49] pry(main)>
```

```
[49] pry(main)> ActiveStorage::Blob.count
   (0.9ms)  SELECT COUNT(*) FROM "active_storage_blobs"
=> 4 // 使用されているデータ4件
[50] pry(main)>
```