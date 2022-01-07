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
