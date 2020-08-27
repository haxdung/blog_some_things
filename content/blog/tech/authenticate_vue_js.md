+++
title = "Authentication với Vue js"
description = "Khi bắt đầu một dự án, việc chúng ta làm đầu tiên, luôn là trang đăng ký đăng nhập, hoặc xử lý luồng authen như thế nào? Lưu lại token như thế nào?"
author = "DungHX"
date = "2020-08-24"
tags = ["authenticate", "vue js"]
categories = ["vue js"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/authenticate_vue.jpg"
  alt = "Authentication vue js image"
  stretch = "horizontal"
+++

Trong mấy bài trước chúng ta đã cùng nhau tạo app todo list với vuejs, vuex và nắm được kiến thức cơ bản về vue js rồi, tới lần này mình sẽ tìm hiểu về làm thế nào để authenticate trong vue js.

Khi bắt đầu một dự án, việc chúng ta làm đầu tiên, luôn là trang đăng ký đăng nhập, hoặc xử lý luồng authen như thế nào? Chúng ta có những câu hỏi sau:

* Lưu lại token như thế nào?
* Làm cách nào để redirect user sau khi authen
* Làm sao để chặn user truy cập một số route nếu chưa authen

Trong bài viết này chúng ra sẽ đi cùng nhau trả lời những câu hỏi này và áp dụng nó vào trong ứng dụng vue, vuex của minh.

Tuy nhiên chúng ta cũng nên nhớ rằng mỗi project vue sẽ sử dụng luồng authen khác nhau. Nên không nên áp dụng cứng nhắc các cách được đề xuất trong bài viết này vào project của mình mà hãy nên cân nhắc lựa chọn hay custom lại để phù hợp với dự án của mình.

Vue.js là một khung JavaScript tiến bộ để xây dựng các ứng dụng ngoại vi.

Để mô tả cho luồng authen chúng ta sẽ implement nó qua chức năng login, chúng ta sẽ có form login và sử dụng axios để gọi ajax

Form login:
```
<template>
 <div>
   <form class="login" @submit.prevent="login">
     <h1>Sign in</h1>
     <label>User name</label>
     <input required v-model="username" type="text" placeholder="Snoopy"/>
     <label>Password</label>
     <input required v-model="password" type="password" placeholder="Password"/>
     <hr/>
     <button type="submit">Login</button>
   </form>
 </div>
</template>
```

Khi người dùng đã điền thông tin đầu vào và click vào Đăng nhập, chúng ta sẽ gọi method login ở trong vue.

```
...
methods: {
 login: function () {
   const { username, password } = this
   this.$store.dispatch(AUTH_REQUEST, { username, password }).then(() => {
     this.$router.push('/')
   })
 }
}
...
```
Vuex sẽ trả về promises. Trong đó action dispatch đuợc coi như là promises. Bây giờ chúng ta có thể handle các lần đăng nhập thành công từ component. Cho phép chúng ta chuyển hướng phù hợp.

### Khai báo Vuex Auth Module

Đầu tiên chúng ta sẽ khởi tạo một state, và đưa vào trong store:

```
const state = {
  token: localStorage.getIte('user-token') || '',
  status: ''
}
```
Trong đó:
* token: lúc khởi tạo nếu có trong localStorage thì lấy giá trị này
* status: trạng thái của network request (loading, success, error)

Sau đó sẽ phải khai báo getters để lấy đc giá trị state này trong store

```
const getters = {
  isAuthenticated: state => !!state.token,
  authStatus: state => state.status,
}
```

Actions xử lý login, logout:

```
const actions = {
  [AUTH_REQUEST]: ({commit, dispatch}, user) => {
    // Dùng promise để sau khi login thành công
    // chúng ta có thể redirect user đi đến trang khác
    return new Promise((resolve, reject) => {
      commit(AUTH_REQUEST)
      axios({url: 'auth', data: user, method: 'POST' })
        .then(resp => {
          const token = resp.data.token
                localStorage.setItem('user-token', token)
                // thêm header authorization:
                axios.defaults.headers.common['Authorization'] = token
                commit(AUTH_SUCCESS, resp)
                dispatch(USER_REQUEST)
                resolve(resp)
        })
      .catch(err => {
        commit(AUTH_ERROR, err);
        // nếu có lỗi, xóa hết token đang có trên trình duyệt
        localStorage.removeItem('user-token');
        reject(err);
      })
    })
  },
  [AUTH_LOGOUT]: ({commit, dispatch}) => {
    return new Promise((resolve, reject) => {
      commit(AUTH_LOGOUT);
      localStorage.removeItem('user-token');
      delete axios.defaults.headers.common['Authorization'];
      resolve();
    })
  }
}
```

Mutations thực hiện update lại store

```
const mutations = {
  [AUTH_REQUEST]: (state) => {
    state.status = 'loading'
  },
  [AUTH_SUCCESS]: (state, token) => {
    state.status = 'success'
    state.token = token
  },
  [AUTH_ERROR]: (state) => {
    state.status = 'error'
  },
}
```


Trạng thái Token đang được khởi tạo bởi giá trị lưu ở cal storage
request yêu cầu xác thực trả về một Promise, sau đó sẽ chuyển hứong sau khi đăng nhập thành công xảy ra.

Trong trường hợp logout sẽ xóa user’s token và chuyển hướng chúng. Nếu bạn cần thực hiện các thay đổi trạng thái Vuex khác, sẽ đc gọi tới hành động AUTH_LOGOUT.

Bạn cũng có thể thêm request action DELETE của mình để xóa user token session của người dùng khi đăng xuất.

### Tự động authen

Hiện tại nếu chúng ta f5 lại, chúng ta vẫn có token bên trong store (do lấy từ localStorage). Tuy nhiên authorization header trong Axios chưa được set

Trong file main.js sẽ như sau:
```
const token = localStorage.getItem('user-token')
if (token) {
  axios.defaults.headers.common['Authorization'] = token
}
```

### Authen Route

Chúng ta có thể cấp quyền và hạn chế người dùng vào một số route mà bạn muốn chỉ khi có token khi mà người dùng đc xác thực hay ko

Ví dụ đã đăng nhập sẽ được vào /account, chưa đăng nhập có thể vào /login và /

```
import store from '../store' // your vuex store

const ifNotAuthenticated = (to, from, next) => {
  if (!store.getters.isAuthenticated) {
    next()
    return
  }
  next('/')
}

const ifAuthenticated = (to, from, next) => {
  if (store.getters.isAuthenticated) {
    next()
    return
  }
  next('/login')
}

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'Home',
      component: Home,
    },
    {
      path: '/account',
      name: 'Account',
      component: Account,
      beforeEnter: ifAuthenticated,
    },
    {
      path: '/login',
      name: 'Login',
      component: Login,
      beforeEnter: ifNotAuthenticated,
    },
  ],
})
```

Ở đây chúng tôi sử dụng navigation guards. Chúng cho phép chúng ta đặt điều kiện truy cập router mà chúng ta sử dụng sẽ dễ dàng cho chúng ta khi kết hợp với Vuex store.

Nếu bạn không muốn sử dụng Vuex, bạn vẫn có thể check có token trong local storage thay vì kiểm tra store getters

### Xử lý các case khác
Các tình huống khác có thể xảy ra như acc token hết hạn, user không đựoc cấp quyền gọi API

Sử dụng Axios, chúng ta có option là interceptors, nó như một bộ lọc, tất cả những response trả về từ network request sẽ đi qua đây. Chúng sẽ kiểm tra tất cả request nào trả về lỗi HTTP 401, dispatch ra logout action

```
created: function () {
  axios.interceptors.response.use(undefined, function (err) {
    return new Promise(function (resolve, reject) {
      if (err.status === 401 && err.config && !err.config.__isRetryRequest) {
      // if you ever get an unauthorized, logout the user
        this.$store.dispatch(AUTH_LOGOUT)
      // you can also redirect to /login if needed !
      }
      throw err;
    });
  });
}
```

### Tổng kết
- Logic authenticate sẽ đuợc tách ra rõ ràng từ ứng dụng và các lib khác
- chúng ta sẽ ko cần thiết phải truyền tokens cho mọi api khi gọi
- Handle được tất cả các lệnh gọi API chưa được authenticate
- Chúng ta có xác thực tự động
- Chúng ta có thể hạn chế truy cập vào router

Tài liệu tham khảo: https://blog.sqreen.com/authentication-best-practices-vue/
https://medium.com/@zitko/structuring-a-vue-project-authentication-87032e5bfe16
