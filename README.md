# Blog サンプル
## 概要

Railsにこれから初めて触れる方を対象にしたチュートリアルです

## チュートリアル
### Railsのひな型を作る

まず、`rails new`を実行し、Railsアプリのひな型を作成します。

```shell
rails new blog
```

次に、作成したRailsアプリのディレクトリへと移動します。

```shell
cd blog
```

### ScaffoldでCRUDを作成

`rails g scaffold` コマンドを使い、ブログに必要な一覧ページや記事作成ページなどを作ります

```shell
rails g scaffold post title:string content:text date:date
```

その後、`rails db:migrate`でマイグレーションを行います

```shell
rails db:migrate
```

あとは`rails s`を実行して、`localhost:3000/posts`にアクセスします

### コメント機能の作成

折角ですので、コメントを投稿できるようにしたいですよね？

まず、コメントを取り扱う`Comment`モデルを作成します。

```shell
rails g model comment content:text post:references
```

マイグレーションを行います

```shell
rails db:migrate
```

`app/models/post.rb`に`Comment`との関連付けを記述します

```ruby:app/models/post.rb
class Post < ApplicationRecord
    has_many :comments
end
```

そして、`config/routes.rb` にてルーティングを設定します

```ruby:config/routes.rb
Rails.application.routes.draw do
　root 'posts#index'
  resources :posts do
    resources :comments, :only => [:create, :destroy]
  end
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

次に、各記事へのコメントフォームを作成していきます。

まずは下記のように`app/views/posts/show.html.erb`を変更します

```erb:app/views/posts/show.html.erb
<p id="notice"><%= notice %></p>

<p>
  <strong>Title:</strong>
  <%= @post.title %>
</p>

<p>
  <strong>Content:</strong>
  <%= @post.content %>
</p>

<p>
  <strong>Date:</strong>
  <%= @post.date %>
</p>

<h2>Comments</h2>
  <div id="comments">
    <%= render @post.comments %>
  </div>

<%= render 'comments/new', post: @post %> 

<%= link_to 'Edit', edit_post_path(@post) %> |
<%= link_to 'Back', posts_path %>
```

変更箇所としてはこの部分になります

```erb
<h2>Comments</h2>
  <div id="comments">
    <%= render @post.comments %>
  </div>

<%= render 'comments/new', post: @post %> 
```

`<%= render @post.comments %>` の部分でコメントを一覧できるviewファイルをパーシャルとして呼び出しています

また、`<%= render 'comments/new', post: @post %> `の部分では新規作成するコメントのviewファイルをパーシャルとして呼び出しています

パーシャルとして呼び出している部分をそれぞれ作っていきます

まず、`app/views/comments/_comment.html.erb` を作成し、下記のようにします。

```erb:app/views/comments/_comment.html.erb
<p><%= comment.content %></p>
<p><%= link_to "Delete", "#{comment.post_id}/comments/#{comment.id}", method: :delete, data: { confirm: 'Are you sure?' } %> 
```

次に、`app/views/comments/_new.html.erb`を作成します

```erb:app/views/comments/_new.html.erb
<%= form_with(model: [ @post, Comment.new ], remote: true) do |form| %>
    Your Comment:<br>
    <%= form.text_area :content, size: '50x20' %><br>
    <%= form.submit %>
<% end %> 
```

これで、コメントの入力フォームが表示されるようになっているはずです

次に、コメントの作成と削除を行うアクションを作成します

`app/controllers/comments_controller.rb`を作成します

```ruby:app/controllers/comments_controller.rb
class CommentsController < ApplicationController
    before_action :set_post

    def create
        @post.comments.create! comments_params
        redirect_to @post
    end

    def destroy
        @post.comments.destroy params[:id]
        redirect_to @post
    end

     private
        def set_post
            @post = Post.find(params[:post_id])
        end

         def comments_params
            params.required(:comment).permit(:content)
        end
end
```

あとは、`rails s` でローカルサーバを建てて実際にコメントが作成&削除できていればOKです。

### リッチなテキストエディターを使う

Rails6では`ActionText`というものが導入されています。
これを使うことで非常に簡単にリッチなテキストエディターを実装することができます。

まずは`rails action_text:install`を実行して、`ActionText`をインストールします。

```bash
rails action_text:install
```

その後、マイグレーションファイルなどが生成されていますのでDBに適用します。

```bash
rails db:migrate
```

マイグレーション後、`Post`モデルを以下のように編集します。

```diff
class Post < ApplicationRecord
+    has_rich_text :content
end
```

その後、`app/views/posts/_form.html.erb`を以下のように編集します。

```diff
<%= form_with(model: post) do |form| %>
  <% if post.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>

      <ul>
        <% post.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= form.label :title %>
    <%= form.text_field :title %>
  </div>

  <div class="field">
    <%= form.label :content %>
+    <%= form.rich_text_area :content %>
+    <%= form.text_area :content %>
  </div>

  <div class="field">
    <%= form.label :date %>
    <%= form.date_select :date %>
  </div>

  <div class="actions">
    <%= form.submit %>
  </div>
<% end %>
```

これでリッチなテキストエディタが使えるようになりました！

## ライセンス
[MITライセンス](../LICENSE)