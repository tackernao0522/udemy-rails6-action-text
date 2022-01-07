## 20 バリデーションを実装する -投稿のタイトル

- `app/models/post.rb`を編集<br>

```
class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximun: 32}
end
```

- `$ bundle exec rails c -s`を実行<br>

```
[2] pry(main)> post = Post.new // enter
   (0.8ms)  BEGIN
=> #<Post:0x00005610b25bd1c8 id: nil, title: nil, created_at: nil, updated_at: nil>
[3] pry(main)>
```

```
[4] pry(main)> post.title // enter
=> nil
[5] pry(main)>
```

```
[6] pry(main)> post.valid?
=> true
[7] pry(main)>
```

```
[8] pry(main)> "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだら" // enter
=> "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだら"
[9] pry(main)>
```

```
[10] pry(main)> "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだら".length // enter
=> 33
[11] pry(main)>
```

```
[12] pry(main)> post.title = "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだら" // enter
=> "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだら"
[13] pry(main)>
```

```
[14] pry(main)> post // enter
=> #<Post:0x00005610b25bd1c8 id: nil, title: "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだら", created_at: nil, updated_at: nil>
[15] pry(main)>
```

```
[15] pry(main)> post.valid? // enter
=> false
[16] pry(main)>
```

```
[16] pry(main)> post.title.length // enter
=> 33
[17] pry(main)>
```

```
[17] pry(main)> post.title.chop // enter (一文字だけ削るメソッド)
=> "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだ"
[18] pry(main)>
```

```
[20] pry(main)> post.title.chop.length // enter
=> 32
[21] pry(main)>
```

```
[22] pry(main)> post.title.length // enter (chopメソッドは非破壊的メソッドである)
=> 33
```

```
[23] pry(main)> post.title // enter
=> "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだら"
[24] pry(main)>
```

```
[26] pry(main)> post.title.chop! (破壊的メソッドになる)
=> "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだ"
[27] pry(main)>
```

```
[28] pry(main)> post.title.length // enter
=> 32
[29] pry(main)>
```

```
[29] pry(main)> post.valid? // enter
=> true
[30] pry(main)>
```

```
[32] pry(main)> post.save! // 永続化する
   (0.7ms)  SAVEPOINT active_record_1
  Post Create (9.7ms)  INSERT INTO "posts" ("title", "created_at", "updated_at") VALUES ($1, $2, $3) RETURNING "id"  [["title", "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだ"], ["created_at", "2022-01-06 08:54:51.925692"], ["updated_at", "2022-01-06 08:54:51.925692"]]
   (1.4ms)  RELEASE SAVEPOINT active_record_1
=> true
[33] pry(main)>
```

```
[34] pry(main)> post // enter
=> #<Post:0x0000560be5bb5480 id: 4, title: "もし高校野球のマネージャーがドラッガーの「マネジメント」を読んだ", created_at: Thu, 06 Jan 2022 08:54:51 UTC +00:00, updated_at: Thu, 06 Jan 2022 08:54:51 UTC +00:00>
[35] pry(main)>
```

```
[37] pry(main)> post = Post.new // enter
=> #<Post:0x0000560be545ddb8 id: nil, title: nil, created_at: nil, updated_at: nil>
[38] pry(main)>
```

```
[39] pry(main)> post.valid? // enter
=> true
[40] pry(main)>
```

```
[40] pry(main)> edit app/models/post.rb // enter
```

- `post.rb`を編集(pry から編集)<br>

```
class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true
end
```

```
[41] pry(main)> reload! // enter 反映させる
Reloading...
=> true
[42] pry(main)>
```

```
[43] pry(main)> post = Post.new
=> #<Post:0x0000560be404e390 id: nil, title: nil, created_at: nil, updated_at: nil>
[44] pry(main)>
```

```
[46] pry(main)> post.title // enter
=> nil
[47] pry(main)>
```

```
[48] pry(main)> post.valid? // enter
=> false
[49] pry(main)>
```

```
[50] pry(main)> post.errors // enter
=> #<ActiveModel::Errors:0x0000560be3513d90
 @base=#<Post:0x0000560be404e390 id: nil, title: nil, created_at: nil, updated_at: nil>,
 @details={:title=>[{:error=>:blank}]},
 @messages={:title=>["can't be blank"]}>
[51] pry(main)>
```

## 21 バリデーション動作をブラウザから確認する

- 参考： https://guides.rubyonrails.org/i18n.html#configure-the-i18n-module <br>

* `config/initializers/locale.rb`を作成<br>

```
# config/initializers/locale.rb

# Where the I18n library should search for translation files
I18n.load_path += Dir[Rails.root.join('lib', 'locale', '*.{rb,yml}')]

# Permitted locales available for the application
I18n.available_locales = [:en, :ja]

# Set default locale to something other than :en
I18n.default_locale = :ja
```

- `config/locales/ja.yml`を作成<br>

```
ja:
  activerecord:
    errors:
      models:
        post:
          attributes:
            title:
              blank: が空です。
              too_long: が%{count}文字を超えています。
    attributes:
      post:
        title: タイトル
```

## 22 バリデーションを実装する - 投稿のWYSIWYGの長さ

+ `$ bundle exec rails c -s`を実行<br>

```
[1] pry(main)> post = Post.find(5)
   (0.7ms)  BEGIN
  Post Load (4.9ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 5], ["LIMIT", 1]]
=> #<Post:0x0000561f059943a8 id: 5, title: "タイトル", created_at: Fri, 07 Jan 2022 08:07:50 UTC +00:00, updated_at: Fri, 07 Jan 2022 08:07:50 UTC +00:00>
[2] pry(main)>
```

```
[4] pry(main)> post.content
  ActionText::RichText Load (5.8ms)  SELECT "action_text_rich_texts".* FROM "action_text_rich_texts" WHERE "action_text_rich_texts"."record_id" = $1 AND "action_text_rich_texts"."record_type" = $2 AND "action_text_rich_texts"."name" = $3 LIMIT $4  [["record_id", 5], ["record_type", "Post"], ["name", "content"], ["LIMIT", 1]]
=> #<ActionText::RichText:0x0000561f03f07698  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 13.9ms | Allocations: 621)

 id: 5,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 5,
 created_at: Fri, 07 Jan 2022 08:07:50 UTC +00:00,
 updated_at: Fri, 07 Jan 2022 08:07:50 UTC +00:00>
[5] pry(main)>
```

```
[7] pry(main)> post.content.class
=> ActionText::RichText(id: integer, name: string, body: text, record_type: string, record_id: integer, created_at: datetime, updated_at: datetime)
[8] pry(main)>
```

```
[10] pry(main)> show-source ActionText::RichText

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
[11] pry(main)>
```

```
[12] pry(main)> post.content.to_plain_text
=> "背景\n\n２０２０年東京オリンピックを開催します。\n\nつきましては、、、\n内容\n内容\n内容\n内容\n内容\n\n敬具"
[13] pry(main)>
```

+ 参考: https://api.rubyonrails.org/v6.0.0/ <br>

```
[15] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length

  MAX_CONTENT_LENGTH = 50

  def validate_content_length
   if  content.to_plain_text.length > MAX_CONTENT_LENGTH
     errors.add(:content, :too_long)
   end
  end
end
```

```
[15] pry(main)> edit config/lacales/ja.yml

ja:
  activerecord:
    errors:
      models:
        post:
          attributes:
            title:
              blank: が空です。
              too_long: が%{count}文字を超えています。
            content:
              too_long: が長いです。
    attributes:
      post:
        title: タイトル
        content: コンテンツ
```

```
[15] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length

  MAX_CONTENT_LENGTH = 50

  def validate_content_length
   if content.to_plain_text.length > MAX_CONTENT_LENGTH
     errors.add(:content, :too_long, max_content_length:  MAX_CONTENT_LENGTH)
   end
  end
end
```

```
[18] pry(main)> edit config/locales/ja.yml

ja:
  activerecord:
    errors:
      models:
        post:
          attributes:
            title:
              blank: が空です。
              too_long: が%{count}文字を超えています。
            content:
              too_long: が%{max_content_length}文字を超えています。
    attributes:
      post:
        title: タイトル
        content: コンテンツ
```

```
[15] pry(main)> edit app/models/post.rb

class Post < ApplicationRecord
  has_rich_text :content

  validates :title, length: {maximum: 32}, presence: true

  validate :validate_content_length

  MAX_CONTENT_LENGTH = 50

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
[20] pry(main)> edit config/locales/ja.yml

ja:
  activerecord:
    errors:
      models:
        post:
          attributes:
            title:
              blank: が空です。
              too_long: が%{count}文字を超えています。
            content:
              too_long: が%{max_content_length - length}文字を超えています。[%{length}/%{max_content_length}]
    attributtes:
      post:
        title: タイトル
        content: コンテンツ
```