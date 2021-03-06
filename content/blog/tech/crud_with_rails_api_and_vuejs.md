+++
title = "Tạo một ứng dụng CRUD với Rails API và Vue.js"
description = "Các bước đơn giản nhất để tạo 1 ứng dụng cũng đơn giản với Rails API và Vue.js"
author = "DungHX"
date = "2020-09-07"
tags = ["rails", "vue js", "ruby"]
categories = ["vue js", "ruby", "rails"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/crud_rails&vue.png"
  alt = "Tạo một ứng dụng CRUD với Rails API và Vue.js"
  stretch = "horizontal"
+++

Xin chào các bạn, đến tháng lại lên. Gần đây mình có tìm hiểu một chút về Rails API và Vuejs. Hôm nay mình cùng các bạn sẽ thực hiện tìm hiểu về cách dùng và cấu trúc một project sử dụng rails và vue js. Nên ở bài viết này mình sẽ chia sẻ các bước đơn giản nhất để tạo 1 ứng dụng cũng đơn giản với Rails API và Vue.js

### Tạo một Rails API

Trước khi bắt đầu thật tốt cho phần này bạn hãy nhớ update ruby version >=2.2.2 và rails version >= 5.

Còn ở trong bài viết của mình sẽ sử dụng ruby 2.5.1, rails 5.2.1 để bắt đầu cho rail api

Trước tiên để tìm hiểu về API mình nghĩ bạn nên hiểu một chút về HTTP và RESTfull API.

RESTful endpoints của ứng dụng như sau:
![](https://images.viblo.asia/a6eef210-7ac6-47d6-807d-7b4c226d248f.png)

**Thiết lập project**

Để tạo một ứng dụng rails api ta chạy câu lệnh sau:
```
rails new todos-api --api -T
```

Trong đó :

 "--api" chúng ta muốn một ứng dụng Rails API, tức là tạo ApplicationController kế thừa ActionController::API thay vì ActionController::Base trong các ứng dụng Rails thông thường.

 "-T" thì đơn giản là để bỏ qua các minitest mặc định mà mình không dùng đến, thay vào đó mình sẽ dùng RSPEC nhưng trong phạm vi bài viết mình sẽ không đề cập đến.


**Tạo model**


Để tạo model chúng ta chạy lệnh sau:
```
rails g model Todo title:string created_by:string status:integer
```
Và sau đó thì migration để tạo bảng:
```
rails db:migrate
```
**Định nghĩa controller**

Tiếp theo là tạo controller cho model Todo

```
rails g controller Todos
```
Sau đó chúng ta sẽ định nghĩa trong routes
```
resources :todos do
    member do
      put "update_status"
    end
  end
```
Giờ thì bắt đầu định nghĩa các function cho controller Todo. Vì ứng dụng của mình khá đơn giản là hiển thị danh sach Todo, thêm sửa xóa todo, update status nên mình sẽ định nghĩa các function như sau:

```
class TodosController < ApplicationController
  before_action :load_todo, except: [:index, :create]

  def index
    @todos = Todo.all
  	json_response(@todos)
  end

  def create
    @todo = Todo.create!(todo_params)
    json_response(@todo, :created)
  end

  def update
    if @todo.update(todo_params)
      json_response(@todo)
    else
      # do something
    end
  end

  def update_status
    if @todo.update status: 'done'
      json_response(@todo)
    else
      # do something
    end
  end

  def destroy
    if @todo.destroy
      json_response(@todo)
    else
      # do something
    end
  end

  private
  def todo_params
    params.permit(:title, :created_by)
  end

  def load_todo
    @todo = Todo.find(params[:id])
  end
end
```

Vì với một API thì dữ liệu trả về không phải dạng html như rails app thông thường mà dữ liệu trả về dạng json. Chúng ta tạo 1 file response.rb
```
#app/controllers/concerns/response.rb
module Response
  def json_response(object, status = :ok)
    render json: object, status: status
  end
end
```

Tuy nhiên nếu chỉ như vậy thì controller của chúng ta sẽ ko thể biết được về sự tồn tại của helpers này, vậy nên chúng ta sẽ including nó trong application controller
```
class ApplicationController < ActionController::API
  include Response
end
```

Ok,vậy là xong rồi đấy, chúng ta đã xong 1 API cực kì đơn giản

### Tạo một Vue project dùng Vue cli

Vừa rồi chúng ta đã tạm viết xong rails API giờ thì sẽ bắt tay vào việc viết view cho nó vì chúng ta không thể hiển thị dữ liệu kiểu json thế kia phải không.

Gọi tắt là Vue (phát âm là /vjuː/, giống như view trong tiếng Anh), Vue.js là một framework linh động dùng để xây dựng giao diện người dùng . Khác với các framework nguyên khối (monolithic), Vue được thiết kế từ đầu theo hướng cho phép và khuyến khích việc phát triển ứng dụng theo từng bước. Khi phát triển lớp giao diện (view layer), người dùng chỉ cần dùng thư viện lõi (core library) của Vue, vốn rất dễ học và tích hợp với các thư viện hoặc dự án có sẵn. Cùng lúc đó, nếu kết hợp với những kĩ thuật hiện đại như SFC (single file components) và các thư viện hỗ trợ, Vue cũng đáp ứng được dễ dàng nhu cầu xây dựng những ứng dụng một trang (SPA - Single-Page Applications) với độ phức tạp cao hơn nhiều.


**Cài đặt Vue-CLI**

Giới thiệu qua một chút về Vue-CLi, thì Vue-CLI là một công cụ giúp xây dựng một ứng dụng mẫu rất nhanh chóng với chỉ vài dòng lệnh

Trước tiên, bạn cần cài đặt Node.js và npm(hoặc yarm trong bài mình sử dụng npm).

Và sau đó là cài Vue-CLI:

`npm install -g vue-cli`

**Xây dựng phần view cho todos app bằng Vue-CLI**

Tạo ứng dụng bằng lệnh:

`vue init webpack vue-todos`

sau đó bạn chạy `npm run dev` để chạy vue project của mình, hãy mở cổng 8080 và kiểm tra kết quả của mình nha!

**implement code**

Như vậy chúng ta đã tạo ra một ứng dụng mẫu Vue.js và có cài đặt sẵn vue-router.

Trong thư mục src có cấu trúc như sau:
* File main.js: đây chính là nơi ứng dụng Vue bắt đầu chạy.
* File App.vue: là component được dùng để render cho trang index.html.
* Thư mục assets: chứa các tài nguyên được chèn vào trang như ảnh....
* components: chứa các component
* router: có các router thực hiện trong ứng dụng.

*Trong file router/index.js thêm:*

```
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Todos from '@/components/Todos'
import AddTodo from '@/components/AddTodo'
import Item from '@/components/Item'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
			path: '/todos',
			name: 'Todos',
			component: Todos
    },
    {
			path: '/add_todo',
			name: 'AddTodo',
			component: AddTodo
    },
    {
			path: '/item',
			name: 'Item',
			component: Item
    }
  ]
})
```

Sau đó tạo components Todo trong thư mục components tạo file Todos.vue

```
<template>
  <div class ="container" style="text-align: left">
    <div class="list-info">
      <p><strong>Completed Tasks:</strong> {{ todos.filter(todo => { return todo.status == 'done' }).length }}</p>
      <p><strong>Pending Tasks:</strong> {{ todos.filter(todo => { return todo.status == 'pending' }).length }}</p>
    </div>

    <h2>Add new todo: </h2>
    <AddTodo v-on:create-todo="CreateTodo" />

    <h2>Todos List:</h2>
    <Item class="col-sm-3"
      v-on:complete-todo="completeTodo"
      v-on:delete-todo="deleteTodo"
      v-on:update-todo="updateTodo"
      v-for="(todo,index) in todos"
      v-bind:todo="todo"
      v-bind:key="index"
    />
  </div>
</template>
<script>
import axios from 'axios';
import AddTodo from './AddTodo';
import Item from './Item';

export default {
  components: {
    AddTodo,
    Item
  },
  data () {
    return {
      todos: [],
    }
  },
  created () {
    axios.get('http://127.0.0.1:3000/todos')
      .then(response => {
        this.todos = response.data
      })
  },
  methods: {
    deleteTodo(todo) {
      const index = this.todos.indexOf(todo);
      axios.delete('http://127.0.0.1:3000/todos/' + todo.id)
        .then(response => {
          this.todos.splice(index, 1)
        });
    },
    updateTodo(todo) {
      axios
        .put(`http://127.0.0.1:3000/todos/${todo.id}`, {
          id: todo.id,
          title: todo.title,
          created_by: todo.created_by
        })
        .then(response => {
        console.log(response);
        })
        .catch(error => {
        console.log(error);
      });
    },
    CreateTodo(title, created_by){
      axios.post('http://localhost:3000/todos', {
            title: title,
            created_by: created_by
          })
          .then(response => {
          })
          .catch(function(error){
            console.log(error);
          });
    },
    completeTodo(todo) {
      axios
        .put(`http://localhost:3000/todos/${todo.id}/update_status`, {
          id: todo.id
        })
        .then(response => {
          console.log(response);
          todo.status = response.data.status;
        })
        .catch(error => {
        console.log(error);
      });
    }
  }
}
</script>

```

Tạo component Item, trong thư mục components tạo file Item.vue

```
<template>
  <div class ="container" style="text-align: left">
    <div class="content" v-show="!isEditing">
      <ul>
        <p>{{todo.id}} </p>
        <p>Title: <strong>{{todo.title}}</strong></p>
        <p>Created by: <strong>{{todo.created_by}}</strong></p>
        <p>Status: <strong>{{todo.status}}</strong></p>
      </ul>
      <ul>
        <span class=" floatd edit icon">
          <button v-on:click="showForm">edit</button>
        </span>
        <span class=" floatd trash icon">
          <button v-on:click="deleteTodo(todo)">delete</button>
        </span>
      </ul>
      <ul>
        <div class="ui bottom attached green basic button" v-show="!isEditing && todo.status=='done'" disabled>
          Completed
        </div>
        <button class="ui bottom attached red basic button" v-show="!isEditing && todo.status=='pending'" v-on:click="completeTodo(todo)">
          Pending
        </button>
      </ul>
    </div>
    <div class="form_edit" v-show="isEditing">
      <div class="ui form">
        <div class="field">
          <label for="">Title</label>
          <input type="text" v-model="todo.title" pattern=".{1,11}" required title="1 to 10 characters">
        </div>
        <div class="field">
          <label for="">created by</label>
          <input type="text" v-model="todo.created_by" pattern=".{1,11}" required title="1 to 10 characters">
        </div>
        <div class="">
          <button class="" v-on:click="updateTodo(todo)">
            update & close
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: ['todo'],
  data() {
    return {
      isEditing: false
    };
  },
  methods: {
    showForm() {
      this.isEditing = true;
    },
    deleteTodo(todo) {
      this.$emit("delete-todo", todo);
    },
    updateTodo(todo) {
      this.isEditing = false;
      this.$emit("update-todo", todo);
    },
    completeTodo(todo) {
      this.$emit("complete-todo", todo);
    }
  }
}
</script>
```
Tạo component AddTodo, trong thư mục components tạo file AddTodo.vue

```
<template>
  <div id='todo'>
  <div class="container">
    <div class="content-container">
      <form v-on:submit.prevent='createTodo'>
        <label for='title'>Title</label>
        <input v-model='title' type='text' pattern=".{1,10}" required title="1 to 10 characters">
        <input v-model='created_by' type='text' pattern=".{1,10}" required title="1 to 10 characters">
        <button type="submit" class='login'>submit</button>
      </form>
    </div>
  </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'AddTodo',
  data () {
    return {
      title: '',
      created_by: ''
    }
  },
  methods: {
    createTodo: function () {
      this.$emit("create-todo", this.title, this.created_by);
      this.title = "";
      this.created_by = "";
    }
  },
}
</script>
```

### Dùng CORS để kết hợp Rails API và Vue.js

CORS (hay nói một cách giông dài là Cross-Origin Resource Sharing) là một kĩ thuật được sinh ra để làm cho việc tương tác giữa client và server được dễ dàng hơn, nó cho phép JavaScript ở một trang web có thể tạo request lên một REST API được host ở một domain khác.

Rails hỗ trợ 1 gem giúp làm điều này đó là gem rack-cors. sau đó bạn hãy vào gem file và add gem đó vào
```
gem 'rack-cors', require: 'rack/cors'
```

Bundle xong chúng ta sẽ thấy trong thư mục config/initializers có file cors.rb, giờ thì vào file cors.rb này để config:

```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
 allow do
  origins 'localhost:8080'

  resource '*',
   headers: :any,
   methods:  %i(get post put patch delete options head)
  end
end
```

Vậy là mình đã xây dựng xong một ứng dụng đơn giản giữa rails api và vue.js với chức năng CRUD model todo và hiển thị số task đã xong và chưa xong, update status của task đó

Cảm ơn bạn đã đọc đến đây ạ , bài của mình có hơi dài xíu =))

Tài liệu tham khảo :

https://scotch.io/tutorials/build-a-restful-json-api-with-rails-5-part-one

https://vuejs.org/

Code sample mình up ở đây cho mn tham khảo nha:

https://github.com/haxdung/rails_vue/
