+++
title = "Components trong VueJs"
description = "Các ứng dụng web ngày nay bao gồm nhiều phần và cách tốt nhất để theo dõi tất cả các phần chuyển động là chia chúng thành các phần nhỏ khác nhau (components)"
author = "DungHX"
date = "2020-09-05"
tags = ["Components", "vue js"]
categories = ["vue js"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/components_in_vue.png"
  alt = "Components trong VueJs"
  stretch = "horizontal"
+++

Components làm cho code có thể sử dụng lại và cho phép chúng ta tuân thủ nguyên tắc D.R.Y . Components là các khối code HTML cơ bản và có thể được sử dụng nhiều lần cho các mục đích khác nhau. Điều này có nghĩa là các Components trong Vue trông giống như các phần tử HTML cơ bản, nhưng chúng có thể config nhiều hơn và do đó, có thể thực hiện nhiều chức năng hơn một phần tử HTML đơn giản. Các Components cũng có thể chứa các Components khác.

Các ứng dụng web ngày nay bao gồm nhiều phần và cách tốt nhất để theo dõi tất cả các phần chuyển động là chia chúng thành các phần nhỏ khác nhau (components), giúp chúng dễ dàng cấu trúc, sử dụng và bảo trì. Vì vậy, cuối cùng, bạn có thể kết thúc với cách dùng components tương tự như thế này cho toàn bộ trang, thực hiện rất nhiều chức năng.

```
<html>
  <head>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.13/vue.js'></script>
  </head>

  <div id='app'>
    <app-header :links="links"></app-header>
    <app-sidebar :items="items"></app-sidebar>
    <app-body></app-body>
    <app-footer></app-footer>
  </div>
</html>
```


Nếu bạn là người maintain, code như thế này rất gọn gàng, và sẽ không mất quá nhiều thời gian để tìm hiểu những phần nào làm gì. Các thành phần trong Vue có thể được tạo theo hai cách, chúng có thể được tạo trong một file riêng biệt, sau đó được import bằng câu lệnh của Es6. Hoặc chúng có thể được registered bên trong file JavaScript và được sử dụng trực tiếp. Bài viết này, chúng ta sẽ tạo ra một component cơ bản lấy một đối tượng người dùng, đưa ra một danh sách và cảnh báo chi tiết người dùng khi mỗi người dùng click vào. Để làm chức năng đó, ta có bước như sau:

1. Tạo Components,
1. Truyền Data tới các components qua Props,
1. Render danh sách,
1. Xây dựng các sự kiện từ component con,
1. Lắng nghe các sự kiện từ component cha
1. Xử lý các sự kiện

**Cài đặt.**

Có hai cách để thiết lập dự án Vue:

1. Sử dụng Webpack build tool
1. Sử dụng Vue qua Vue CDN.

### 1: Định nghĩa  Component.

Components có thể được tạo hoàn toàn trong 1 file riêng biệt hoặc trực tiếp bên trong file JavaScript. Ví dụ này chúng ta sẽ định nghĩa components của chúng ta ngay trong file JavaScript của mình.
Components được registered bằng lệnh Vue.component ('tag-name', options), trong đó tag-name là tên mà bạn muốn component của mình và options là một đối tượng xác định hành vi của component. Điều này làm cho component có thể gọi ở mọi nơi trong file và do đó, giờ đây có thể được sử dụng trong các trường hợp khác nhau.

Hãy bắt đầu bằng một component hiển thị một thông báo trên màn hình, bằng cách gọi user-list. Để làm theo, hãy tạo một file HTML mới như sau:

```
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.13/vue.js'></script>
</head>
<body>

    <div id="app">
      <user-list></user-list>
    </div>

    <script type="text/javascript">

      let userList = Vue.component('user-list', {
      template : '<div>I am a component</div>'

      });

       let app = new Vue({
           el : "#app"

       });

    </script>

</body>
</html>
```

Chúng ta đã tạo một Vue component đặt tên là user-list, trong thẻ html chúng ta đã sử dụng user-list như một thẻ HTML thông thường. Đây là cách bạn xuất component của bạn lên view. Bạn có thể thấy thuộc tính mẫu trong lúc định nghĩa Vue component, điều này chỉ định các thẻ HTML sẽ có output bởi component . Hãy lưu ý rằng một component Vue chỉ có một root element.. Đó là tất cả những gì có để tạo ra một component cơ bản trong Vue.


### 2: Props và Components

Mỗi component Vue hoạt động trong một phạm vi riêng và không nên truy cập dữ liệu từ bên ngoài. Props giúp ta truyền dữ liệu từ một component cha (bên ngoài), đến một component con. Trong trường hợp này, chúng ta sẽ chuyển dữ liệu từ app đến component userList. Nhưng trước khi chúng ta có thể làm điều này, chúng ta phải xác định rõ ràng props mà chúng ta đang mong đợi trong component userList của mình. Thêm một thuộc tính khác vào component userList, gọi nó là props, đây sẽ là một mảng của tất cả các props mà chúng ta đang mong đợi được chuyển đến component userList. Đặt nội dung của thuộc tính props là ['users'].sau đó chúng ta sẽ thay đổi thuộc tính mẫu và xóa tất cả nội dung của div thay thế chúng bằng {{users}}.

Ngoài ra, trong tệp HTML chính, hãy thêm một thuộc tính mới là users vào thẻ <user-list> và đặt giá trị thành users = "list of users". bây h code của chúng ta sẽ ntn:

```
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.13/vue.js'></script>
</head>
<body>

    <div id="app">
      <user-list users="list of users"></user-list>
    </div>

    <script type="text/javascript">

      let userList = Vue.component('userList', {
      template : '<div>{{users}}</div>',
      props: ['users']

      });

       let app = new Vue({
           el : "#app"

       });

    </script>

</body>
</html>
```


Như chúng ta có thể thấy,component của chúng ta đã trở nên linh hoạt hơn, giờ đây dữ liệu có thể được truyền từ component cha sang nó, sử dụng users attribute.

Điều này không có nghĩa là chỉ các chuỗi có thể được truyền trong props, các biến cũng có thể được truyền qua, sử dụng thuộc tính V-bind Vue. Trong ứng dụng Vue này chúng ta sẽ xác định một thuộc tính dữ liệu và chuyển vào biến sẽ được sử dụng bởi component Vue. data attribute sẽ như sau:

```
data(){
    return{
      allUsers : [
        {
          name : 'John Doe',
          about : 'Really nice guy'
        },
        {
          name : 'Jane Dean',
          about: 'Loves eggs'
        },
        {
          name : 'Clark Kent',
          about: 'Not your everyday reporter'
        }
      ]
    }
  }
```


 Điều này về cơ bản chỉ trả về một mảng gồm ba đối tượng với hai keys, name và about. Để chuyển list users mới được xác định của chúng ta vào component, chúng ta chỉ cần thêm thuộc tính v-bind: users vào component đó và truyền name của mảng cho nó, do đó chúng ta có <user-list v-bind: users = "allUsers "> </user-list>. Tiền tố v-bind: cho Vue biết rằng chúng ta muốn tự động liên kết các props users với một biến và không truyền trực tiếp vào một chuỗi ký tự.

```
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.13/vue.js'></script>
</head>
<body>

    <div id="app">
      <user-list v-bind:users="allUsers"></user-list>
    </div>

    <script type="text/javascript">

      let userList = Vue.component('userList', {
      template : '<div>{{users}}</div>',
      props: ['users']

      });

       let app = new Vue({
           el : "#app",

           data(){
            return{
              allUsers : [
                {
                  name : 'John Doe',
                  about : 'Really nice guy'
                },
                {
                  name : 'Jane Dean',
                  about: 'Loves eggs'
                },
                {
                  name : 'Clark Kent',
                  about: 'Not your everyday reporter'
                }
              ]
            }
           }

       });

    </script>

</body>
</html>
```
 Remember we said earlier that we want our component to be able to list all users passed to it. To do this, we need to perform List Rendering, using the v-for directive. The directive is used to render a list of items based on an array.
The syntax is like this:

Chúng tôi muốn component của chúng ta có thể liệt kê tất cả users được truyền cho nó. Để làm điều này, chúng ta cần thực hiện render danh sách, sử dụng lệnh v-for. Lệnh được sử dụng để hiển thị danh sách dựa trên một mảng:

```
<li v-for="item in items"></li>
```

### 3: Xử lý sự kiện Click

Vì chúng ta muốn các components có thể được sử dụng lại, chúng ta sẽ không xử lý sự kiện click bên trong components, thay vào đó, chúng ta sẽ trả lại sự kiện cho components chính, sẽ sử dụng thông qua payload để thực hiện bất kỳ hành động nào mà nó yêu cầu. Ưu điểm của việc này là chúng ta có thể sử dụng cùng một components cho nhiều mục đích khác nhau.

Chúng ta sẽ làm điều này bằng cách làm cho user-list component có sự kiện khi một item được click và chúng ta sẽ xử lý sự kiện này trên component cha. sẽ bắt sự kiện onclick vào phần tử <li>, chúng tôi thực hiện điều này trong Vue bằng cách thêm thuộc tính @click. Sự kiện nhấp chuột này sẽ gọi một phương thức nội bộ và truyền attribute vào method.
```
<li v-for="user in users" @click="emitClicked(user.about)">
  {{user.name}}
</li>
```
Bạn có thể thấy ở trên, có một phương thức được truyền cho sự kiện click handler, được gọi là phương thức emitClicky, chúng ta sẽ định nghĩa phương thức này bằng cách thêm thuộc tính phương thức vào component Vue .
```
methods : {
  emitClicked(data){
      this.$emit('item-clicked',data)
}
```

### 4: Lắng nghe sự kiện.


Cách dễ nhất để nghe một sự kiện trong component cha là sử dụng thuộc tính v-on, vì vậy chúng tôi có thể dễ dàng lắng nghe sự kiện bằng cách thêm thuộc tính v-on: item-click vào thẻ HTML <user-list>.

```
<user-list v-bind:users="allUsers" v-on:item-clicked="alertData"></user-list>
```

Từ đoạn code trên, chúng ta có thể thấy rằng có một method mới gọi là alertData, Phương thức này là thứ xử lý payload(data) được truyền ra từ component con khi nó phát ra sự kiện. Chúng ta sẽ định nghĩa method alertData bên trong component chính bằng cách thêm thuộc tính của phương thức.
```
methods:
{
  alertData(data)
  {
    alert(data)
  }
}
```

method này chỉ đơn giản là sử dụng alert để hiển thị dữ liệu đã được truyền từ component con.

```
 <!DOCTYPE html>
    <html>
    <head>
        <title></title>
        <script src='https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.13/vue.js'></script>
    </head>
    <body>

        <div id="app">
          <user-list v-bind:users="allUsers" v-on:item-clicked="alertData"></user-list>
        </div>

        <script type="text/javascript">

          let userList = Vue.component('userList', {
          template : `
            <ul>
              <li v-for="user in users" @click="emitClicked(user.about)">
                {{user.name}}
              </li>
            </ul>
          `,

          props: ['users'],

          methods : {
            emitClicked(data){

              this.$emit('item-clicked',data)

            }
          }

          });

           let app = new Vue({
               el : "#app",

               data(){
                return{
                  allUsers : [
                    {
                      name : 'John Doe',
                      about : 'Really nice guy'
                    },
                    {
                      name : 'Jane Dean',
                      about: 'Loves eggs'
                    },
                    {
                      name : 'Clark Kent',
                      about: 'Not your everyday reporter'
                    }
                  ]
                }
               },

               methods:
               {
                  alertData(data)
                  {
                    alert(data)
                  }
               }

           });

        </script>

    </body>
    </html>
```

Khả năng sử dụng lại của component này nằm ở chỗ v-on: click-item có thể dùng đc với các methods khác nhau và tạo ra outputs khác nhau, do đó, component user-list có thể được sử dụng lại trên ứng dụng.


Tài liệu tham khảo:

   https://dev.to/faradayyg/components-in-vuejs-j5g

   https://vi.vuejs.org/v2/guide/components.html
