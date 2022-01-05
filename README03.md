## 15 モデルの相関について見ていく - posts テーブル編

- `db/schema.rb`には全てのテーブルの定義が書かれている<br>

## 16 モデルの相関について見ていく - action_text_rich_texts テーブル編

- posts テーブルには rich_text のデータが関連づく<br>

- Post モデルに書かれた`has_rich_text :content`1 対 1 によって関連づけられている<br>

- `$ bundle exec rails c -s`でコンソールを立ち上げる<br>

```
root@54e0fd7e43f1:/app# bundle exec rails c -s
/app/vendor/bundle/gems/pry-byebug-3.8.0/lib/pry-byebug/control_d_handler.rb:5: warning: control_d_handler's arity of 2 parameters was deprecated (eval_string, pry_instance). Now it gets passed just 1 parameter (pry_instance)
Loading development environment in sandbox (Rails 6.0.4.4)
Any modifications you make will be rolled back on exit
[1] pry(main)> show-source Post.has_rich_text // 実行してみる
```

```
From: /app/vendor/bundle/gems/actiontext-6.0.4.4/lib/action_text/attribute.rb:8:
Owner: ActionText::Attribute::ClassMethods
Visibility: public
Signature: has_rich_text(name)
Number of lines: 17

def has_rich_text(name)
  class_eval <<-CODE, __FILE__, __LINE__ + 1
    def #{name}
      rich_text_#{name} || build_rich_text_#{name}
    end

    def #{name}=(body)
      self.#{name}.body = body
    end
  CODE

  has_one :"rich_text_#{name}", -> { where(name: name) },
    class_name: "ActionText::RichText", as: :record, inverse_of: :record, autosave: true, dependent: :destroy

  scope :"with_rich_text_#{name}", -> { includes("rich_text_#{name}") }
  scope :"with_rich_text_#{name}_and_embeds", -> { includes("rich_text_#{name}": { embeds_attachments: :blob }) }
end
[2] pry(main)>
```

```
[2] pry(main)> Pry.config.pager
=> true // pagerが有効になっている
[3] pry(main)> Pry.config.pager = false = false // enterで実行してみる
[4] pry(main)> Pry.config.pager
=> false // falseになる
```

- pager が出なくなる<br>

```
[8] pry(main)> show-source Post.has_rich_text

From: /app/vendor/bundle/gems/actiontext-6.0.4.4/lib/action_text/attribute.rb:8:
Owner: ActionText::Attribute::ClassMethods
Visibility: public
Signature: has_rich_text(name)
Number of lines: 17

def has_rich_text(name)
  class_eval <<-CODE, __FILE__, __LINE__ + 1
    def #{name}
      rich_text_#{name} || build_rich_text_#{name}
    end

    def #{name}=(body)
      self.#{name}.body = body
    end
  CODE

  has_one :"rich_text_#{name}", -> { where(name: name) },
    class_name: "ActionText::RichText", as: :record, inverse_of: :record, autosave: true, dependent: :destroy

  scope :"with_rich_text_#{name}", -> { includes("rich_text_#{name}") }
  scope :"with_rich_text_#{name}_and_embeds", -> { includes("rich_text_#{name}": { embeds_attachments: :blob }) }
end
[9] pry(main)> Post.find(3) // enter
```

```
[3] pry(main)> Post.find(3)
   (1.0ms)  BEGIN
  Post Load (6.0ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
=> #<Post:0x0000556477298a20 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[4] pry(main)> post // enter
```

```
[5] pry(main)> post
=> #<Post:0x000055647732ff38 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[6] pry(main)> post.content // enter
```

```
[7] pry(main)> post.content
  ActionText::RichText Load (21.6ms)  SELECT "action_text_rich_texts".* FROM "action_text_rich_texts" WHERE "action_text_rich_texts"."record_id" = $1 AND "action_text_rich_texts"."record_type" = $2 AND "action_text_rich_texts"."name" = $3 LIMIT $4  [["record_id", 3], ["record_type", "Post"], ["name", "content"], ["LIMIT", 1]]
=> #<ActionText::RichText:0x0000556475c75dd8  ActiveStorage::Blob Load (8.8ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 164.4ms | Allocations: 3293)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 481.1ms | Allocations: 16154)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 3,
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[8] pry(main)> post.content.class // enter
```

```
[9] pry(main)> post.content.class
=> ActionText::RichText(id: integer, name: string, body: text, record_type: string, record_id: integer, created_at: datetime, updated_at: datetime)
[10] pry(main)> post.content.class.table_name // enter
```

```
[11] pry(main)> post.content.class.table_name
=> "action_text_rich_texts"
[12] pry(main)> content = post.content // enter
```

```
[12] pry(main)> content = post.content
=> #<ActionText::RichText:0x0000556475c75dd8  ActiveStorage::Blob Load (0.9ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 1.3ms | Allocations: 499)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 18.1ms | Allocations: 2588)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post", // Postクラスのこと
 record_id: 3, // post.idとリレーションしている
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[13] pry(main)>
```

- `bundle exec rails g model article`を実行<br>

- `bundle exec rails db:migrate`を実行<br>

- `app/models/article.rb`を編集<br>

```
class Article < ApplicationRecord
  has_rich_text :statement
end
```

- `bundle exec rails c -s`を実行<br>

```
[1] pry(main)> article = Article.create // enter
   (0.8ms)  BEGIN
   (0.5ms)  SAVEPOINT active_record_1
  Article Create (15.6ms)  INSERT INTO "articles" ("created_at", "updated_at") VALUES ($1, $2) RETURNING "id"  [["created_at", "2022-01-04 05:07:49.440873"], ["updated_at", "2022-01-04 05:07:49.440873"]]
   (0.7ms)  RELEASE SAVEPOINT active_record_1
=> #<Article:0x000055a531820b28 id: 1, created_at: Tue, 04 Jan 2022 05:07:49 UTC +00:00, updated_at: Tue, 04 Jan 2022 05:07:49 UTC +00:00>
[2] pry(main)> article // enter
=> #<Article:0x000055a531820b28 id: 1, created_at: Tue, 04 Jan 2022 05:07:49 UTC +00:00, updated_at: Tue, 04 Jan 2022 05:07:49 UTC +00:00>
[3] pry(main)> article.statement // enter
```

```
[3] pry(main)> article.statement
  ActionText::RichText Load (3.9ms)  SELECT "action_text_rich_texts".* FROM "action_text_rich_texts" WHERE "action_text_rich_texts"."record_id" = $1 AND "action_text_rich_texts"."record_type" = $2 AND "action_text_rich_texts"."name" = $3 LIMIT $4  [["record_id", 1], ["record_type", "Article"], ["name", "statement"], ["LIMIT", 1]]
=> #<ActionText::RichText:0x00005651e1813e20 id: nil, name: "statement", body: nil, record_type: "Article", record_id: 1, created_at: nil, updated_at: nil>
[4] pry(main)> statement = article.statement // enter
```

```
[4] pry(main)> statement = article.statement
=> #<ActionText::RichText:0x00005651e1813e20 id: nil, name: "statement", body: nil, record_type: "Article", record_id: 1, created_at: nil, updated_at: nil>
[5] pry(main)> statement.save! // enter
```

```
[5] pry(main)> statement.save!
   (0.4ms)  SAVEPOINT active_record_1
  ActionText::RichText Create (10.8ms)  INSERT INTO "action_text_rich_texts" ("name", "record_type", "record_id", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["name", "statement"], ["record_type", "Article"], ["record_id", 1], ["created_at", "2022-01-04 05:16:59.232818"], ["updated_at", "2022-01-04 05:16:59.232818"]]
  Article Update (6.8ms)  UPDATE "articles" SET "updated_at" = $1 WHERE "articles"."id" = $2  [["updated_at", "2022-01-04 05:16:59.254976"], ["id", 1]]
   (1.0ms)  RELEASE SAVEPOINT active_record_1
=> true
[6] pry(main)> statement // enter
```

```
[6] pry(main)> statement
=> #<ActionText::RichText:0x00005651e1813e20
 id: 3, // saveするとidが付与されている
 name: "statement",
 body: nil,
 record_type: "Article",
 record_id: 1,
 created_at: Tue, 04 Jan 2022 05:16:59 UTC +00:00,
 updated_at: Tue, 04 Jan 2022 05:16:59 UTC +00:00>
[7] pry(main)> statement.record // enter
```

```
[7] pry(main)> statement.record
=> #<Article:0x00005651e32d6cd0 id: 1, created_at: Tue, 04 Jan 2022 05:14:08 UTC +00:00, updated_at: Tue, 04 Jan 2022 05:16:59 UTC +00:00>
[8] pry(main)> Post.find(3).content // enter
```

```
[9] pry(main)> Post.find(3).content
  Post Load (5.4ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ActionText::RichText Load (6.1ms)  SELECT "action_text_rich_texts".* FROM "action_text_rich_texts" WHERE "action_text_rich_texts"."record_id" = $1 AND "action_text_rich_texts"."record_type" = $2 AND "action_text_rich_texts"."name" = $3 LIMIT $4  [["record_id", 3], ["record_type", "Post"], ["name", "content"], ["LIMIT", 1]]
=> #<ActionText::RichText:0x00005651e2bd6110  ActiveStorage::Blob Load (4.3ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 104.3ms | Allocations: 3292)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 276.7ms | Allocations: 16144)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 3,
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[10] pry(main)> Post.find(3).content.record // enter
```

```
[12] pry(main)> Post.find(3).content.record
  Post Load (0.7ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ActionText::RichText Load (0.5ms)  SELECT "action_text_rich_texts".* FROM "action_text_rich_texts" WHERE "action_text_rich_texts"."record_id" = $1 AND "action_text_rich_texts"."record_type" = $2 AND "action_text_rich_texts"."name" = $3 LIMIT $4  [["record_id", 3], ["record_type", "Post"], ["name", "content"], ["LIMIT", 1]]
=> #<Post:0x00005651e33af058 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[13] pry(main)> show-source ActionText::RichText
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

  belongs_to :record, polymorphic: true, touch: true // ポリモーフィックを使っている
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

## 17 モデルの相関について見ていく - active_storage_attachmentsテーブル編

```
[2] pry(main)> show-source ActionText::RitchText

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
[14] pry(main)> show-source ActionText::RichText.has_many_attached // enter
```

```
[14] pry(main)> show-source ActionText::RichText.has_many_attached

From: /app/vendor/bundle/gems/activestorage-6.0.4.4/lib/active_storage/attached/model.rb:66:
Owner: ActiveStorage::Attached::Model::ClassMethods
Visibility: public
Signature: has_many_attached(name, dependent:?)
Number of lines: 49

def has_many_attached(name, dependent: :purge_later)
  generated_association_methods.class_eval <<-CODE, __FILE__, __LINE__ + 1
    def #{name}
      @active_storage_attached ||= {}
      @active_storage_attached[:#{name}] ||= ActiveStorage::Attached::Many.new("#{name}", self)
    end

    def #{name}=(attachables)
      if ActiveStorage.replace_on_assign_to_many
        attachment_changes["#{name}"] =
          if Array(attachables).none?
            ActiveStorage::Attached::Changes::DeleteMany.new("#{name}", self)
          else
            ActiveStorage::Attached::Changes::CreateMany.new("#{name}", self, attachables)
          end
      else
        if Array(attachables).any?
          attachment_changes["#{name}"] =
            ActiveStorage::Attached::Changes::CreateMany.new("#{name}", self, #{name}.blobs + attachables)
        end
      end
    end
  CODE

  has_many :"#{name}_attachments", -> { where(name: name) }, as: :record, class_name: "ActiveStorage::Attachment", inverse_of: :record, dependent: :destroy do
    def purge
      each(&:purge)
      reset
    end

    def purge_later
      each(&:purge_later)
      reset
    end
  end
  has_many :"#{name}_blobs", through: :"#{name}_attachments", class_name: "ActiveStorage::Blob", source: :blob

  scope :"with_attached_#{name}", -> { includes("#{name}_attachments": :blob) }

  after_save { attachment_changes[name.to_s]&.save }

  after_commit(on: %i[ create update ]) { attachment_changes.delete(name.to_s).try(:upload) }

  ActiveRecord::Reflection.add_attachment_reflection(
    self,
    name,
    ActiveRecord::Reflection.create(:has_many_attached, name, nil, { dependent: dependent }, self)
  )
end
[15] pry(main)> post = Post.find(3) // enter
```

```
[15] pry(main)> post = Post.find(3)
  Post Load (1.2ms)  SELECT "posts".* FROM "posts" WHERE "posts"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
=> #<Post:0x000056446334bae0 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[16] pry(main)> post // enter
```

```
[19] pry(main)> post
=> #<Post:0x000056446334bae0 id: 3, title: "First Post", created_at: Sun, 02 Jan 2022 02:32:53 UTC +00:00, updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[20] pry(main)> post = post.content // enter
```

```
[21] pry(main)> content = post.content
=> #<ActionText::RichText:0x00005644636742d0  ActiveStorage::Blob Load (0.8ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 1.5ms | Allocations: 499)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 22.2ms | Allocations: 2586)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 3,
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[22] pry(main)> content // enter
```

```
[23] pry(main)> content
=> #<ActionText::RichText:0x00005644636742d0  ActiveStorage::Blob Load (0.6ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  Rendered active_storage/blobs/_blob.html.erb (Duration: 1.4ms | Allocations: 499)
  Rendered vendor/bundle/gems/actiontext-6.0.4.4/app/views/action_text/content/_layout.html.erb (Duration: 18.9ms | Allocations: 2586)

 id: 2,
 name: "content",
 body: #<ActionText::Content "<div class=\"trix-conte...">,
 record_type: "Post",
 record_id: 3,
 created_at: Sun, 02 Jan 2022 02:32:54 UTC +00:00,
 updated_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[24] pry(main)> content.embeds_attachments // enter
```

```
[26] pry(main)> content.embeds_attachments
=>   ActiveStorage::Attachment Load (29.1ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = $1 AND "active_storage_attachments"."record_type" = $2 AND "active_storage_attachments"."name" = $3  [["record_id", 2], ["record_type", "ActionText::RichText"], ["name", "embeds"]]
[#<ActiveStorage::Attachment:0x0000564463fb9a70
  id: 1,
  name: "embeds",
  record_type: "ActionText::RichText",
  record_id: 2,
  blob_id: 1,
  created_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>]
[27] pry(main)> attachments = content.embeds_attachments // enter
```

```
[30] pry(main)> attachments = content.embeds_attachments
=> [#<ActiveStorage::Attachment:0x0000564463fb9a70
  id: 1,
  name: "embeds",
  record_type: "ActionText::RichText",
  record_id: 2,
  blob_id: 1,
  created_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>]
[31] pry(main)> attachment = attachments.first // enter
```

```
[34] pry(main)> attachment = attachments.first
=> #<ActiveStorage::Attachment:0x0000564463fb9a70 id: 1, name: "embeds", record_type: "ActionText::RichText", record_id: 2, blob_id: 1, created_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[35] pry(main)> attachment // enter
```

```
[35] pry(main)> attachment
=> #<ActiveStorage::Attachment:0x0000564463fb9a70 id: 1, name: "embeds", record_type: "ActionText::RichText", record_id: 2, blob_id: 1, created_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[36] pry(main)> show-source ActiveStorage::Attachment // enter
```

```
[36] pry(main)> show-source ActiveStorage::Attachment

From: /app/vendor/bundle/gems/activestorage-6.0.4.4/app/models/active_storage/attachment.rb:5
Class name: ActiveStorage::Attachment
Number of monkeypatches: 4. Use the `-a` option to display all available monkeypatches
Number of lines: 41

class ActiveStorage::Attachment < ActiveRecord::Base
  self.table_name = "active_storage_attachments"

  belongs_to :record, polymorphic: true, touch: true // ★
  belongs_to :blob, class_name: "ActiveStorage::Blob"

  delegate_missing_to :blob

  after_create_commit :analyze_blob_later, :identify_blob
  after_destroy_commit :purge_dependent_blob_later

  # Synchronously deletes the attachment and {purges the blob}[rdoc-ref:ActiveStorage::Blob#purge].
  def purge
    delete
    blob&.purge
  end

  # Deletes the attachment and {enqueues a background job}[rdoc-ref:ActiveStorage::Blob#purge_later] to purge the blob.
  def purge_later
    delete
    blob&.purge_later
  end

  private
    def identify_blob
      blob.identify
    end

    def analyze_blob_later
      blob.analyze_later unless blob.analyzed?
    end

    def purge_dependent_blob_later
      blob&.purge_later if dependent == :purge_later
    end


    def dependent
      record.attachment_reflections[name]&.options[:dependent]
    end
end
[37] pry(main)> attachment // enter
```

```
[37] pry(main)> attachment
=> #<ActiveStorage::Attachment:0x0000564463fb9a70 id: 1, name: "embeds", record_type: "ActionText::RichText", record_id: 2, blob_id: 1, created_at: Sun, 02 Jan 2022 02:37:03 UTC +00:00>
[38] pry(main)> attachment.class.table_name // enter
```

```
[38] pry(main)> attachment.class.table_name
=> "active_storage_attachments"
[39] pry(main)>
```

## 18 モデルの相関について見ていく - active_storage_blobsテーブル編

```
[41] pry(main)> show-source ActionText::RichText.has_many_attached // enter

From: /app/vendor/bundle/gems/activestorage-6.0.4.4/lib/active_storage/attached/model.rb:66:
Owner: ActiveStorage::Attached::Model::ClassMethods
Visibility: public
Signature: has_many_attached(name, dependent:?)
Number of lines: 49

def has_many_attached(name, dependent: :purge_later)
  generated_association_methods.class_eval <<-CODE, __FILE__, __LINE__ + 1
    def #{name}
      @active_storage_attached ||= {}
      @active_storage_attached[:#{name}] ||= ActiveStorage::Attached::Many.new("#{name}", self)
    end

    def #{name}=(attachables)
      if ActiveStorage.replace_on_assign_to_many
        attachment_changes["#{name}"] =
          if Array(attachables).none?
            ActiveStorage::Attached::Changes::DeleteMany.new("#{name}", self)
          else
            ActiveStorage::Attached::Changes::CreateMany.new("#{name}", self, attachables)
          end
      else
        if Array(attachables).any?
          attachment_changes["#{name}"] =
            ActiveStorage::Attached::Changes::CreateMany.new("#{name}", self, #{name}.blobs + attachables)
        end
      end
    end
  CODE

  has_many :"#{name}_attachments", -> { where(name: name) }, as: :record, class_name: "ActiveStorage::Attachment", inverse_of: :record, dependent: :destroy do
    def purge
      each(&:purge)
      reset
    end

    def purge_later
      each(&:purge_later)
      reset
    end
  end
  has_many :"#{name}_blobs", through: :"#{name}_attachments", class_name: "ActiveStorage::Blob", source: :blob // ★

  scope :"with_attached_#{name}", -> { includes("#{name}_attachments": :blob) }

  after_save { attachment_changes[name.to_s]&.save }

  after_commit(on: %i[ create update ]) { attachment_changes.delete(name.to_s).try(:upload) }

  ActiveRecord::Reflection.add_attachment_reflection(
    self,
    name,
    ActiveRecord::Reflection.create(:has_many_attached, name, nil, { dependent: dependent }, self)
  )
end
[42] pry(main)> content.embeds_blobs // enter
```

```
[44] pry(main)> content.embeds_blobs
=>   ActiveStorage::Blob Load (10.5ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" INNER JOIN "active_storage_attachments" ON "active_storage_blobs"."id" = "active_storage_attachments"."blob_id" WHERE "active_storage_attachments"."record_id" = $1 AND "active_storage_attachments"."record_type" = $2 AND "active_storage_attachments"."name" = $3  [["record_id", 2], ["record_type", "ActionText::RichText"], ["name", "embeds"]]
[#<ActiveStorage::Blob:0x0000564463ee19b8
  id: 1,
  key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
  filename: "001.jpg",
  content_type: "image/jpeg",
  metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
  byte_size: 3629588,
  checksum: "13znRaP8NLezkHEv5lVLfA==",
  created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>]
[45] pry(main)> content.embeds_blobs.count // enter
```

```
[47] pry(main)> content.embeds_blobs.count
   (16.5ms)  SELECT COUNT(*) FROM "active_storage_blobs" INNER JOIN "active_storage_attachments" ON "active_storage_blobs"."id" = "active_storage_attachments"."blob_id" WHERE "active_storage_attachments"."record_id" = $1 AND "active_storage_attachments"."record_type" = $2 AND "active_storage_attachments"."name" = $3  [["record_id", 2], ["record_type", "ActionText::RichText"], ["name", "embeds"]]
=> 1 // ★
[48] pry(main)> content.embeds_blobs.first // enter
```

```
[48] pry(main)> blob = content.embeds_blobs.first
=> #<ActiveStorage::Blob:0x0000564463ee19b8
 id: 1,
 key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
 filename: "001.jpg",
 content_type: "image/jpeg",
 metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
 byte_size: 3629588,
 checksum: "13znRaP8NLezkHEv5lVLfA==",
 created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>
[49] pry(main)> blob // enter
```

```
[50] pry(main)> blob
=> #<ActiveStorage::Blob:0x0000564463ee19b8
 id: 1,
 key: "ke8rd2cmqmn16sba5ty2jio6gmbj",
 filename: "001.jpg",
 content_type: "image/jpeg",
 metadata: {"identified"=>true, "width"=>5015, "height"=>3343, "analyzed"=>true},
 byte_size: 3629588,
 checksum: "13znRaP8NLezkHEv5lVLfA==",
 created_at: Sun, 02 Jan 2022 02:36:39 UTC +00:00>
[51] pry(main)>
```

+ `localターミナルで`<br>

```
groovy@groovy-no-MacBook-Pro udemy-rails6-action-text % ls storage/ke/8r/ke8rd2cmqmn16sba5ty2jio6gmbj
storage/ke/8r/ke8rd2cmqmn16sba5ty2jio6gmbj
groovy@groovy-no-MacBook-Pro udemy-rails6-action-text % open storage/ke/8r/ke8rd2cmqmn16sba5ty2jio6gmbj
groovy@groovy-no-MacBook-Pro udemy-rails6-action-text %
```

```
[53] pry(main)> blob.byte_size
=> 3629588
[54] pry(main)>  blob.byte_size / (1024 * 1000.0) // enter
```

```
[54] pry(main)> blob.byte_size / (1024 * 1000.0)
=> 3.54451953125
[55] pry(main)> Digest::MD5.file('storage/ke/8r/ke8rd2cmqmn16sba5ty2jio6gmbj') // enter
```

```
[55] pry(main)> Digest::MD5.file('storage/ke/8r/ke8rd2cmqmn16sba5ty2jio6gmbj')
=> #<Digest::MD5: d77ce745a3fc34b7b390712fe6554b7c>
[56] pry(main)> Digest::MD5.file('storage/ke/8r/ke8rd2cmqmn16sba5ty2jio6gmbj').base64digest // enter
```

```
[56] pry(main)> Digest::MD5.file('storage/ke/8r/ke8rd2cmqmn16sba5ty2jio6gmbj').base64digest
=> "13znRaP8NLezkHEv5lVLfA==" // checksumと同じ
[57] pry(main)>
```