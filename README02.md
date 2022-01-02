# セクション3: アプリケーション開発編

## 6. Action Textに必要なマイグクレーションを実行する

+ `$ docker-compose exec web ./bin/rails -T`でタスクが閲覧できる<br>

+ `$ bundle exec rake action_text:install`を実行<br>

+ `$ bunle exec rake db:migrate`を実行<br>

## 7. image_processingをインストール

+ `Gemfile`の `gem 'image_processing', '~> 1.2'`をコメントアウトを解除して`$ bundle install`する<br>

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

## Strong Parametersについてのおさらい

+ `app/controllers/posts_controller.rb`を編集<br>

```
class PostsController < ApplicationController
  before_action :set_post, only: %i[ show edit update destroy ]

  # GET /posts or /posts.json
  def index
    @posts = Post.all
  end

  # GET /posts/1 or /posts/1.json
  def show
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

  # GET /posts/1/edit
  def edit
  end

  # POST /posts or /posts.json
  def create
    @post = Post.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to post_url(@post), notice: "Post was successfully created." }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /posts/1 or /posts/1.json
  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to post_url(@post), notice: "Post was successfully updated." }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /posts/1 or /posts/1.json
  def destroy
    @post.destroy

    respond_to do |format|
      format.html { redirect_to posts_url, notice: "Post was successfully destroyed." }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def post_params
      params.require(:post).permit(:title, :content) // 編集
    end
end
```

## 11 viewにRichTextの表示領域を設定する

+ `app/views/posts/_form.html.erb`を編集<br>

```
<%= form_with(model: post, local: true) do |form| %>
  <% if post.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>

      <ul>
        <% post.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>

  <div class="field">
    <%= form.rich_text_area :content %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

+ `app/views/posts/show.html.erb`を編集<br>

```
<p id="notice"><%= notice %></p>

<p>
  <strong>Title:</strong>
  <%= @post.title %>
</p>

<%= @post.content %> // 追記

<%= link_to 'Edit', edit_post_path(@post) %> |
<%= link_to 'Back', posts_path %>
```

## 12 pry-railsをinstallしてデバック

+ `Gemfile`を編集<br>

```
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.6.3'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 6.0.0'
# Use postgresql as the database for Active Record
gem 'pg', '>= 0.18', '< 2.0'
# Use Puma as the app server
gem 'puma', '~> 3.11'
# Use SCSS for stylesheets
gem 'sass-rails', '~> 5'
# Transpile app-like JavaScript. Read more: https://github.com/rails/webpacker
gem 'webpacker', '~> 4.0'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  gem 'pry' // 追記
end

group :development do
  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '>= 3.0.5', '< 3.2'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

+ `$ bundle install`を実行<br>

+ `app/controllers/posts_controller.rb`を編集<br>

```
class PostsController < ApplicationController
  before_action :set_post, only: %i[ show edit update destroy ]

  # GET /posts or /posts.json
  def index
    binding.pry // 追記
    @posts = Post.all
  end

  # GET /posts/1 or /posts/1.json
  def show
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

  # GET /posts/1/edit
  def edit
  end

  # POST /posts or /posts.json
  def create
    @post = Post.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to post_url(@post), notice: "Post was successfully created." }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /posts/1 or /posts/1.json
  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to post_url(@post), notice: "Post was successfully updated." }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /posts/1 or /posts/1.json
  def destroy
    @post.destroy

    respond_to do |format|
      format.html { redirect_to posts_url, notice: "Post was successfully destroyed." }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def post_params
      params.require(:post).permit(:title, :content)
    end
end
```

`コンソール`<br>
```
udemy-rails6-action-text-web-1  | From: /app/app/controllers/posts_controller.rb:6 PostsController#index:
udemy-rails6-action-text-web-1  |
udemy-rails6-action-text-web-1  |     5: def index
udemy-rails6-action-text-web-1  |  => 6:   binding.pry
udemy-rails6-action-text-web-1  |     7:   @posts = Post.all
udemy-rails6-action-text-web-1  |     8: end
udemy-rails6-action-text-web-1  |
```

+ `$ docker attach --help`で確認<br>
```
Usage:  docker attach [OPTIONS] CONTAINER

Attach local standard input, output, and error streams to a running container

Options:
      --detach-keys string   Override the key sequence for detaching a container
      --no-stdin             Do not attach STDIN
      --sig-proxy            Proxy all received signals to the process (default true)
```

+ `$ docker-compose ps --help`で確認<br>
```
Usage:  docker compose ps [SERVICE...]

List containers

Options:
  -a, --all                  Show all stopped containers (including those created by the run command)
      --format string        Format the output. Values: [pretty | json] (default "pretty")
  -q, --quiet                Only display IDs
      --services             Display services
      --status stringArray   Filter services by status. Values: [paused | restarting | removing | running | dead | created | exited]
```

+ `$ docker-compose ps -q web`を実行して出力されたIDをコピーしておく<br>
```
6e019216f7cdf367848af1c9600ed836ea085b9401ada636d7de6662275f0dc8
```

+ `$ docker attach 6e019216f7cdf367848af1c9600ed836ea085b9401ada636d7de6662275f0dc8` を実行<br>
```
// Enterを押すとirbのようなものが立ち上がる
[1] pry(#<PostsController>)>
[2] pry(#<PostsController>)>
```

+ pryコンソールに` whereami`と入力しEnterしてみる<br>

```
[1] pry(#<PostsController>)>
[2] pry(#<PostsController>)> whereami

From: /app/app/controllers/posts_controller.rb:6 PostsController#index:

    5: def index
 => 6:   binding.pry
    7:   @posts = Post.all
    8: end
```

+ pryコンソールに`params`と入力しEnterしてみる<br>

```
[3] pry(#<PostsController>)> params
=> <ActionController::Parameters {"controller"=>"posts", "action"=>"index"} permitted: false>
[4] pry(#<PostsController>)>
```

+ pryコンソールに`request`と入力しEnterしてみる<br>

```
[5] pry(#<PostsController>)> request
=> #<ActionDispatch::Request:0x0000561829b5cdf8
 @env=
  {"rack.version"=>[1, 3],
   "rack.errors"=>#<IO:<STDERR>>,
   "rack.multithread"=>true,
   "rack.multiprocess"=>false,
   "rack.run_once"=>false,
   "SCRIPT_NAME"=>"",
   "QUERY_STRING"=>"",
   "SERVER_PROTOCOL"=>"HTTP/1.1",
   "SERVER_SOFTWARE"=>"puma 3.12.6 Llamas in Pajamas",
   "GATEWAY_INTERFACE"=>"CGI/1.2",
   "REQUEST_METHOD"=>"GET",
   "REQUEST_PATH"=>"/posts",
   "REQUEST_URI"=>"/posts",

<page break> --- Press enter to continue ( q<enter> to break ) --- <page break>
```

+ pryコンソールに`Rails.env`と入力しEnterしてみる<br>

```
[6] pry(#<PostsController>)> Rails.env
=> "development"
[7] pry(#<PostsController>)>
```

+ pryコンソールに`show-source Rails`と入力しEnterしてみる<br>

```
[8] pry(#<PostsController>)> show-source Rails

From: /app/vendor/bundle/gems/railties-6.0.4.4/lib/rails.rb:27
Module name: Rails
Number of monkeypatches: 3. Use the `-a` option to display all available monkeypatches
Number of lines: 93

module Rails
  extend ActiveSupport::Autoload

  autoload :Info
  autoload :InfoController
  autoload :MailersController
  autoload :WelcomeController

  class << self
    @application = @app_class = nil

    attr_writer :application
    attr_accessor :app_class, :cache, :logger
    def application

<page break> --- Press enter to continue ( q<enter> to break ) --- <page break>
```

+ `$ exitしてコントローラのbinding.pryを消しておく`<br>

+ `$ ls bin/docker-compose-attach`を実行<br>
```
bin/docker-compose-attach
```

+ `$ cat bin/docker-compose-attach`を実行<br>
```
#!/usr/bin/env bash

docker-compose up -d && docker attach $(docker-compose ps -q $1)
```

+ `$ docker compose down`でコンテナーを切断する<br>

+ `$ ./bin/docker-compose-attach web`を実行<br>
```
[+] Running 3/3
 ⠿ Network udemy-rails6-action-text_default  Created                                                                                                                              0.3s
 ⠿ Container udemy-rails6-action-text_db_1   Started                                                                                                                              1.4s
 ⠿ Container udemy-rails6-action-text_web_1  Started                                                                                                                              8.6s
=> Booting Puma
=> Rails 6.0.4.4 application starting in development
=> Run `rails server --help` for more startup options
Puma starting in single mode...
* Version 3.12.6 (ruby 2.6.3-p62), codename: Llamas in Pajamas
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

+ 上記の方法だとこの状態でpryと通信ができる<br>

## 13 pry-railsをインストールしてrails consoleでもpryする

+ `$ bundle exec rails console`を実行<br>
```
root@4f521a846a47:/app# bundle exec rails console
Loading development environment (Rails 6.0.4.4)
irb(
```

```
// syntaxエラーになってしまう
irb(main):001:0> show-souce Rails
Traceback (most recent call last):
SyntaxError ((irb):1: syntax error, unexpected tCONSTANT, expecting do or '{' or '(')
show-souce Rails
           ^~~~~
```

+ `Gemfile`を編集<br>

```
source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '2.6.3'

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem 'rails', '~> 6.0.0'
# Use postgresql as the database for Active Record
gem 'pg', '>= 0.18', '< 2.0'
# Use Puma as the app server
gem 'puma', '~> 3.11'
# Use SCSS for stylesheets
gem 'sass-rails', '~> 5'
# Transpile app-like JavaScript. Read more: https://github.com/rails/webpacker
gem 'webpacker', '~> 4.0'
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem 'jbuilder', '~> 2.7'
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem 'bootsnap', '>= 1.4.2', require: false

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  gem 'pry' // 削除
  gem 'pry-rails' // 追記
end

group :development do
  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '>= 3.0.5', '< 3.2'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

+ `$ bundle install`を実行<br>

+ `$ docker compose down`を実行<br>

+ `$ ./bin/docker-compose-attach web`を実行<br>

+ `$ bundle exec rails console`を実行するとpryが立ち上がる<br>
```
root@6d02f62a5376:/app# bundle exec rails console
Loading development environment (Rails 6.0.4.4)
[1] pry(main)>
```

+ `pry(main)> show-source Rails`を実行してみる<br>

```
root@aeaf745b9400:/app# bundle exec rails c
Loading development environment (Rails 6.0.4.4)
[1] pry(main)> show-source Rails

From: /app/vendor/bundle/gems/railties-6.0.4.4/lib/rails.rb:27
Module name: Rails
Number of monkeypatches: 3. Use the `-a` option to display all available monkeypatches
Number of lines: 93

module Rails
  extend ActiveSupport::Autoload

  autoload :Info
  autoload :InfoController
  autoload :MailersController
  autoload :WelcomeController

  class << self
    @application = @app_class = nil

    attr_writer :application
    attr_accessor :app_class, :cache, :logger

<page break> --- Press enter to continue ( q<enter> to break ) --- <page break>
```
