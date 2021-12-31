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