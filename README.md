> master ブランチが初期状態になっている。
> このブランチから切って学習を進めることができる。

---

参考メモ : https://hackmd.io/KDMj08X7QIqkTcs37rEWBQ?view

参考ソースコード : https://github.com/Farstep/todo_app

参考動画 1/4 : https://www.youtube.com/watch?v=yD0QotED6f8

---

### 1. コンテナ立ち上げ
```
docker compose up
```

### 2. yarn で 必要なパッケージをインストール
```
docker compose exec web yarn add react-router-dom axios styled-components react-icons react-toastify react react-dom 
```

メモの通り
> - react-router-dom：Reactでのroutingの実現。
> - axios：サーバとのHTTP通信を行う。
> - styled-components：CSS in JS のライブラリ
> - react-icons：Font Awesomeなどのアイコンが簡単に利用できるライブラリ

---

加えて以下を追加してインストールしてある
> - react 
> - react-dom


---
---
---

### | ステップ１（model & table & data）
```
docker compose exec web rails g model todo name is_completed:boolean
```

マイグレーションファイルに null false、boolean の初期値のfalse を加筆
```ruby
class CreateTodos < ActiveRecord::Migration[7.1]
  def change
    create_table :todos do |t|
      t.string :name, null: false
      t.boolean :is_completed, default: false, null: false

      t.timestamps
    end
  end
end

```

マイグレーション実行
```shell
docker compose exec web rails db:migrate
```


初期データ
[ config/seed.rb ]
```
SAMPLE_TODOS = [
  {
    name: 'Going around the world',
  },
  {
    name: 'graduating from college'
  },
  {
    name: 'publishing a book',
  }
]

SAMPLE_TODOS.each do |todo|
  Todo.create(todo)
end

```

反映
```shell
docker compose exec web rails db:seed
```

### | ステップ２（site#index）

コントローラ & ビュー 生成
```shell
docker compose exec web rails g controller site index
```
site/index.html.erb を 編集
```erb
<div id="root"></div>
```

### | ステップ３（todos_controller）
```
docker compose exec web rails g controller api::v1::todos
```

```ruby
class Api::V1::TodosController < ApplicationController
  def index
    todos = Todo.order(updated_at: :desc)
    render json: todos
  end

  def show
    todo = Todo.find(params[:id])
    render json: todo
  end

  def create
    todo = Todo.new(todo_params)
    if todo.save
      render json: todo
    else
      render json: todo.errors, status: 422
    end
  end

  def update
    todo = Todo.find(params[:id])
    if todo.update(todo_params)
      render json: todo
    else
      render json: todo.errors, status: 422
    end
  end

  def destroy
    if Todo.destroy(params[:id])
      head :no_content
    else
      render json: { error: "Failed to destroy" }, status: 422
    end
  end

  def destroy_all
    if Todo.destroy_all
      head :no_content
    else
      render json: { error: "Failed to destroy" }, status: 422
    end
  end

  private

  def todo_params
    params.require(:todo).permit(:name, :is_completed)
  end
end
```
ルーティング
```ruby
root to: redirect('/todos')

get 'todos', to: 'site#index'
get 'todos/new', to: 'site#index'
get 'todos/:id/edit', to: 'site#index'

namespace :api do
  namespace :v1 do
    delete '/todos/destroy_all', to: 'todos#destroy_all'
    resources :todos, only: %i[index show create update destroy]
  end
end
```
### | ステップ４（application_controller）
```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :null_session
end
```

### | ステップ5 ~~turbolinksの無効化）~~
=> 参考メモでは trubo-links を無効化しているが、Rails7系では無効化してはマウントされなかったのでとばす。

### | ステップ6  ~~（app/views/application.html.erb）~~

これも、参考メモとは違う手法をとる。

```diff
  // Entry point for the build script in your package.json
  import "@hotwired/turbo-rails"
  import "./controllers"
+ import "./react/index"
```

### | ステップ7 （componentsフォルダの作成）

1. [ app/javascript/react ] ディレクトリを作成
2. その中に [ index.jsx ] を作成

```
mkdir -p app/javascript/react
```
```
touch app/javascript/react/index.jsx
```

[ react/index.js ]
```jsx
import React from "react";
import { createRoot } from "react-dom/client";

const container = document.getElementById("root");

if (container) {
  const root = createRoot(container);
  root.render(
    // <React.StrictMode>
    <>
      <h1>hello react</h1>
    </>
    // </React.StrictMode> 
  );
}
```
jsをビルド
```
docker compose exec web yarn js
```
---

##### ビューのマウント、ここまで

---
---
---

