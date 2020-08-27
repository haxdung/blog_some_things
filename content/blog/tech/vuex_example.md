+++
title = "Vuex và ví dụ đơn giản"
description = "Vuejs là một trong những framework JavaScript tốt nhất và nhiều người cho rằng Vue sẽ dần thay thế cho Angular và React trong tương lai."
author = "DungHX"
date = "2020-08-23"
tags = ["vue js", "vuex"]
categories = ["vue js", "vuex"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/vuex.png"
  alt = "vuex image"
  stretch = "horizontal"
+++

Hôm nay bài viết này mình sẽ giới thiệu cho các bạn về Vuex! Một trong những framework mạnh mẽ nhất hiện nay! Vuejs là một trong những framework JavaScript tốt nhất và nhiều người cho rằng Vue sẽ dần thay thế cho Angular và React trong tương lai. Vuejs không mới hơn hay phổ biến hơn so với những frameworks khác nhưng nó vẫn sở hữu những đặc điểm tạo nên sự khác biệt cho nó.

Trong cộng đồng React khi ko thể không nhắc tới 2 cái tên là FLux và Redux, chúng đc sử dụng trong hầu hết các app Js. Còn khi nói với Vue js, người ta sẽ nghĩ ngay tới Vuex. Trong bài này mọi người hãy cùng mình tìm hiểu về Vuex để hiểu follow hoạt động của vuex ntn, rồi start một project todos app sử dụng vuex để hiểu hết bản chất của nó!

## 1: Đầu tiên chúng ta nên hiểu Vue-x là gì

Vue.js là làm việc với các component, tuy nhiên từ trước đến giờ chúng ta chưa có cách nào giúp chia sẻ trạng thái giữa các component này, hoặc có nhưng mã nguồn rất khó duy trì đặc biệt với các dự án lớn. Từ đó Vuex ra đời giúp quản lý trạng thái các component trong Vue.js, nó là nơi lưu trữ tập trung cho tất cả các component trong một ứng dụng, với nguyên tắc trạng thái chỉ có thể được thay đổi theo kiểu có thể dự đoán

Vuex hoạt động theo mô hình "Luồng dữ liệu một chiều" với các thành phần sau:

* State – Trạng thái, là nơi khởi nguồn để thực hiện ứng dụng.
* View – Khung nhìn, là các khai báo ánh xạ với trạng thái.
* Action: Hành động, là những cách thức làm trạng thái thay đổi phản ứng lại các nhập liệu của người dùng từ View.

![abc](https://images.viblo.asia/c9cd4cff-5e48-4873-bd0c-2f77719c5c87.png)

Vuex nhìn thấy tại sao không đưa các trạng thái được chia sẻ của các component ra và quản lý chúng trong một bộ máy toàn cục, và đó chính là lý do cho sự ra đời của Vuex. Trong đó, các component trở thành các view và các component có thể truy xuất trạng thái hoặc trigger các hành động. Với cách thức này, mã nguồn có cấu trúc và dễ dàng duy trì.

![abc](https://images.viblo.asia/5a404a2d-7b2d-4b4d-ae14-91cac35b823a.png)

### store

Trung tâm của mọi ứng dụng Vuex là kho chứa (store), một kho chứa đơn giản là nơi khai báo, lưu trữ các trạng thái ứng dụng. Store có thể tác động lại các Vue component lấy trạng thái từ đó, trạng thái sẽ được cập nhật khi có thay đổi một cách hiệu quả. Các trạng thái trong store không thể thay đổi trực tiếp, chỉ có một cách duy nhất để thay đổi trạng thái là phải commit.

### State

Vuex sử dụng một cây trạng thái duy nhất, đối tượng này sẽ chứa tất các trạng thái của ứng dụng, như vậy bạn chỉ có duy nhất một store cho mỗi ứng dụng, điều này làm cho việc xác định các trạng thái là dễ dàng.  Để sử dụng các trạng thái trong Vue component, chúng ta sẽ lấy các trạng thái và trả về trong thuộc tính computed của component:

### Getter – bộ lọc trạng thái

Đôi khi chúng ta cần lấy các trạng thái dựa vào việc tính toán, lọc bỏ các trạng thái được cung cấp bởi store. Nếu nhiều component cần làm điều này, chúng ta có thể định nghĩa getter trong store để thực hiện, khi đó chúng ta có thể truy xuất các trạng thái đã được lọc này.

### Mutations – thay đổi trạng thái

Trạng thái không thể thay đổi trực tiếp mà chỉ được thay đổi thông qua commit, Vuex mutation tương tự như các sự kiện, mỗi mutation có kiểu chuỗi và một handler. Handler function là nơi chúng ta thực hiện các thay đổi trạng thái và nó cần được truyền vào tham số đầu tiên là state.

### Action – hành động

Action cũng tương tự như mutation, tuy nhiên có một vài điểm khác biệt. Thay vì thay đổi trạng thái, action commit các thay đổi. Action có thể chứa các hoạt động không đồng bộ.

=> Vậy những điểm mạnh của vuex là gì :
Vuex giúp cho việc quản lý dự án của chúng ta trở nên hiệu quả. Nó cân bằng giữa tốc độ và hiệu năng của dự án. Có thể chúng ta sẽ cảm thấy code vuex khá là dài dòng, phức tạp so với code thuần vue js và có vẻ không mang lại hiệu quả gì. Nhưng Khi bạn code một SPA (Single-Page App) tầm trung đến lớn, vuex có lẽ là sự lựa chọn hoàn toàn sáng suốt cho chúng ta. Chả cần phải chần chừ suy nghĩ, vuex là lựa chọn tốt nhất rồi đó =))

Để chứng minh điều đấy, chúng ta sẽ bắt tay luôn vào code thôi nào =))

## 2: Ví dụ đơn giản với vuex

Đầu tiên hãy kiểm tra cài đặt node js và npm trên máy bạn

```$ nodejs -v
v9.4.0
$ npm -v
5.6.0
```

Cài đặt VueCLI:

```html
$ npm install -g vue-cli
```

Khởi tạo project:

```html
$ vue init webpack todo-task-vuex
```

Cài đặt package vuex:

```html
$ npm install --save vuex
```

Vậy là đã xong phần khởi tạo

Vuex có 5 Core Concepts: State, Getters, Mutations, Actions, Modules. Project này mình sẽ chỉ sử dụng 3 Core Concepts là State, Mutations và Actions.

### Coding

**Trong src tạo folder có tên store và xây dựng file index.js trong strore:**

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import { mutations } from './mutations'
import actions from './actions'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    todos: []
  },
  actions,
  mutations,
})

```

Nên tách mutations, actions thành file js riêng để dễ quản lý

* mutations.js

```javascript
export const mutations = {
  addTodo (state, todo) {
    state.todos.push(todo)
  },

  removeTodo (state, todo) {
    state.todos.splice(state.todos.indexOf(todo), 1)
  },

  editTodo (state, { todo, text = todo.text, done = todo.done }) {
    const index = state.todos.indexOf(todo)

    state.todos.splice(index, 1, {
      ...todo,
      text,
      done
    })
  }
}

```

* actions.js

```javascript
export default {
  getTask ({commit}, task) {
    commit('getTask', task)
  },
  addTask ({commit}) {
    commit('addTask')
  },
  editTask ({commit}, task) {
    commit('editTask', task)
  },
  removeTask ({commit}, task) {
    commit('removeTask', task)
  },
  completeTask ({commit}, task) {
    commit('completeTask', task)
  },
  clearTask ({commit}) {
    commit('clearTask')
  }
}

```

* Đừng quên import store trong file main.js

```javascript
import Vue from 'vue'
import App from './App'
import store from './store/index';

new Vue({
  el: '#app',
  store,
  render: h => h(App)
})

```

* Sau đó trong src tạo folder có tên components và tạo component có tên TodoItem.vue

```html
<template>
  <li class="todo" :class="{ completed: todo.done, editing: editing }">
    <div class="view">
      <input class="toggle"
        type="checkbox"
        :checked="todo.done"
        @change="toggleTodo(todo)">
      <label v-text="todo.text" @dblclick="editing = true"></label>
      <button class="destroy" @click="removeTodo(todo)"></button>
    </div>
    <input class="edit"
      v-show="editing"
      v-focus="editing"
      :value="todo.text"
      @keyup.enter="doneEdit"
      @keyup.esc="cancelEdit"
      @blur="doneEdit">
  </li>
</template>

<script>
import { mapActions } from 'vuex'
export default {
  name: 'Todo',
  props: ['todo'],
  data () {
    return {
      editing: false
    }
  },
  directives: {
    focus (el, { value }, { context }) {
      if (value) {
        context.$nextTick(() => {
          el.focus()
        })
      }
    }
  },
  methods: {
    ...mapActions([
      'editTodo',
      'toggleTodo',
      'removeTodo'
    ]),
    doneEdit (e) {
      const value = e.target.value.trim()
      const { todo } = this
      if (!value) {
        this.removeTodo(todo)
      } else if (this.editing) {
        this.editTodo({
          todo,
          value
        })
        this.editing = false
      }
    },
    cancelEdit (e) {
      e.target.value = this.todo.text
      this.editing = false
    }
  }
}
</script>
```

**Import các components vào App.vue cho ứng dụng:**

```html
<template>
  <div id="app" class="container">
    <section class="todoapp">
      <!-- header -->
      <header class="header">
        <h1>TO DO LIST</h1>
        <input class="new-todo"
          autofocus
          autocomplete="off"
          placeholder="What needs to be done?"
          @keyup.enter="addTodo">
      </header>
      <!-- main section -->
      <section class="main" v-show="todos.length">
        <input class="toggle-all" id="toggle-all"
          type="checkbox"
          :checked="allChecked"
          @change="toggleAll(!allChecked)">
        <label for="toggle-all"></label>
        <ul class="todo-list">
          <TodoItem
            v-for="(todo, index) in filteredTodos"
            :key="index"
            :todo="todo"
          />
        </ul>
      </section>
      <!-- footer -->
      <footer class="footer" v-show="todos.length">
        <span class="todo-count">
          <strong>{{ remaining }}</strong>
          {{ remaining | pluralize('item') }} left
        </span>
        <ul class="filters">
          <li v-for="(val, key) in filters">
            <a :href="'#/' + key"
              :class="{ selected: visibility === key }"
              @click="visibility = key">{{ key | capitalize }}</a>
          </li>
        </ul>
        <button class="clear-completed"
          v-show="todos.length > remaining"
          @click="clearCompleted">
          Clear completed
        </button>
      </footer>
    </section>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import TodoItem from './components/TodoItem.vue'
const filters = {
  all: todos => todos,
  active: todos => todos.filter(todo => !todo.done),
  completed: todos => todos.filter(todo => todo.done)
}
export default {
  components: { TodoItem },
  data () {
    return {
      visibility: 'all',
      filters: filters
    }
  },
  computed: {
    todos () {
      return this.$store.state.todos
    },
    allChecked () {
      return this.todos.every(todo => todo.done)
    },
    filteredTodos () {
      return filters[this.visibility](this.todos)
    },
    remaining () {
      return this.todos.filter(todo => !todo.done).length
    }
  },
  methods: {
    ...mapActions([
      'toggleAll',
      'clearCompleted'
    ]),
    addTodo (e) {
      const text = e.target.value
      if (text.trim()) {
        this.$store.dispatch('addTodo', text)
      }
      e.target.value = ''
    }
  },
  filters: {
    pluralize: (n, w) => n === 1 ? w : (w + 's'),
    capitalize: s => s.charAt(0).toUpperCase() + s.slice(1)
  }
}
</script>

<style src="todomvc-app-css/index.css"></style>
```

* Thêm css cho project

Để làm cho project đẹp hơn mình đã sử dụng todomvc-app-css, link hướng dẫn mình để dưới nha =))

[todomvc-app-css](https://github.com/tastejs/todomvc-app-css)

### Done! Mở terminal ngay và luôn và xem thành quả

```html
$ npm run dev
```

![aa](https://images.viblo.asia/67c0bf6c-1363-48f4-992a-84ddb63ffaaa.png)

Qua bài viết chắc rằng chúng ta cũng có thêm những hiểu viết về Vuex và áp dụng hiệu quả cho các dự án của mình.

Link github:  [todo_task](https://github.com/haxdung-1663/Todo_Task_Vuex)


Tài liệu tham khảo : [vuex](https://vuex.vuejs.org)
