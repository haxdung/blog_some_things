+++
title = "Tìm hiểu cơ bản về React"
description = "React là gì? Tại sao nên sử dụng React? State và Lifecycle trong React"
author = "DungHX"
date = "2020-09-20"
tags = ["react", "state", "lifecycle"]
categories = ["react"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/react.jpg"
  alt = "react image"
  stretch = "horizontal"
+++

# 1: React là gì?

Một thư viện Javascript dùng để xây dựng giao diện người dùng. Ta có thể hiểu React giống như "phần V ở trong mô hình MVC". Trên thực tế, các framework như Angular, Backbone, Ember... cũng đã vốn cung cấp cho chúng ta đầy đủ về tầng View. Điều này dẫn đến câu hỏi: "Tại sao chúng ta lại cần đến React để thay thế cho phần V?"
Câu trả lời đó là: React không hẳn thực sự thay thế các views mà nó làm tăng cường view bằng cách cho phép chúng ta tạo ra các UI component có tính sử dụng lại cao (như Tab bars, comment box, pop up, modals, lists, button, table v.v...). Ý tưởng chủ đạo của React là: "Bạn có thể tạo cho chính bạn các phần tử HTML có khả năng customized một cách dễ dàng".

# 2: Tại sao nên sử dụng React?

React cho phép các lập trình viên tạo ra các Web application lớn có thể thay đổi data mà không phải reload lại trang. Mục đích chủ yếu của React là nhanh, dễ dàng scale up và đơn giản. React chỉ động ở phần interface trong một ứng dụng.

### Simplicity - Tính Đơn giản

Với đặc tính thiết kế dựa trên component với các vòng đời (lifecycle) được định nghĩa rõ ràng và sử dụng Plain Javascript khiến cho React trở nên rất dễ dàng sử dụng. React sử dụng syntax đặc biệt là JSX cho phép chúng ta viết trộn HTML và JavsScript cùng nhau. Ta không bắt buộc phải sử dụng JSX, developer vẫn có thể viết Plain JavaScript nhưng JSX sẽ dễ dàng sử dụng hơn rất nhiều.

### Easy to learn - Dễ dàng học

Bất cứ ai với kiến thức lập trình cơ bản cũng có thể dễ dàng hiểu được React trong khi đó Angular và Ember thường được biết đến là DSL - Domain specific Language nên có rất nhiều tính đặc thù riêng khó học hơn. Với React thì bạn chỉ cần biết các kiến thức cơ bản về CSS và HTML. React vô cùng đơn giản do nó chỉ thao tác với tầng View.


### Native Approach - Dễ dàng tiếp cận với native

React có thể được dùng để tạo ra các ứng dụng điện thoại sử dụng React Native. Hỗ trợ việc tái sử dụng code và được coi là thư viện 'learn's once write anywhere' - ' học một lần viết ở khắp nơi'. Với native thì bạn có thể hiểu là React Native cung cấp cho bạn khả năng viết các native app có thể chạy được trên cả Android lẫn iOS.


### Reusable Components - Khả năng tái sử dụng code

React cung cấp cho ta cấu trúc dựa trên component. Các component ở đây đóng vai trò như những miếng lego. Ta có thể bắt đầu với các component nhỏ như button, checkbox, dropdown v.v... và tiến đến tạo các wrapper component để bọc, chứa các component nhỏ hơn đó. Và rồi lại tiếp tục viết các component bọc to hơn cho đến khi tiến tới component root.
Mỗi component sẽ quyết định ứng dụng  được render như thế nào và mỗi chúng đều có một logic riêng của mình. Phương pháp này đã cho thấy tính hiệu quả tuyệt vời. Ta có thể sử dụng lại các component ở bất kì đâu cần thiết. Và kết quả là ta có một ứng dụng có sự đồng nhất, thống nhất trong giao diện, thao tác. Việc sử dụng lại code khiến cho việc maitain vô cùng dễ dàng và việc phát triển thêm cho ứng dụng lại càng dễ hơn.

### Great Developer Tools - Developer Tools tiện nhất
Khi code React có hai tool sẽ thường xuyên được sử dụng mà ta cần để ý đến:  **React Developer Tools** và **Redux Developer Tools**. Cả hai tool này đều có thể được cài đặt thông qua Chrome extensions.
**React Developer Tools** là công cụ tuyệt vời để inspect React component theo đúng cây nó được sắp xếp và dễ dàng để quan sát được `props` và `states` hiện tại của component đó. Component lựa chọn sẽ được highlight phía bên trái, vị trí của nó trong cây và props của nó được hiển thị bên cột phải.

Nếu bạn đang sử dụng library Redux thì chắc chắn nên xem, tìm kếm **Redux Developer Tools**. Bạn có thể quan sát được các action được dispatch, các state lưu trong store và theo dõi các thay đổi trong store. Đồng thời ta có thể dispatch một action hoặc thay đổi store và theo dõi thay đổi được áp dụng lên page ngay lập tức


# 3: State and Lifecycle

## 3.1. State

##### 3.1.1 Vậy state là gì ?

- State là một **object** javascript.
- Cả state và props đều là **công cụ để lưu trữ data** ở trong component nhưng state được quản lý ở ngay trong component, ngược lại với props là được truyền vào (giống như parameter của function).

##### 3.1.2 Tại sao phải dùng state ?

- Như bài trước chúng ta đã biết đó là những component được tạo ra bằng cách sử dụng function và nhận vào props rồi dùng props đó để render ra jsx component (stateless component).
- Nhưng trong thực tế thì component không chỉ làm những công việc đơn giản như thế, mà chúng ta còn cần phải xử lý dữ liệu và hiển thị nội trong component, ví dụ như một component form hoặc các component có thể tái sử dụng được. Trong các ví dụ tiếp theo chúng ta sẽ tìm hiểu.

##### 3.1.3 Sử dụng state như thế nào

- Để sử dụng state ta bắt buộc phải sử dụng statefull component, ta sẽ khai báo statefull component như sau:

```
import React from 'react';

export default class TodoForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            textContent: '',
        };
        this.handleChange = this.handleChange.bind(this);
    }

    handleChange(e) {
        this.setState({ textContent: e.target.value });
    }

    render() {
        const { textContent } = this.state;
        return (
            <div>
                <input
                    type='text'
                    value={textContent}
                    onChange={this.handleChange}
                />
            </div>
        );
    }
}
```

- Vậy là ta đã có một component **TodoForm** và component này có thể import vào bất cứ đâu để sử dụng rồi.

## 3.2. Lifecycle

![State and Lifecycle](/img/2020/tech/react_state_lifecycle.jpg)

##### 3.2.1 Lifecycle là gì ?

- Đó là vòng đời của một component, từ khi nó được khởi tạo cho đến khi được mount vào DOM rồi cho đến khi nó được unmount và destroy khỏi DOM. Tất cả các component của React đều tuân theo vòng đời này.
- React đã cung cấp sẵn cho chúng ta các hook để chúng ta có thể gọi được vào các lifecycle methods này, theo đó chúng ta có thể control được component của chúng ta từ lúc sinh ra cho đến khi bị xoá đi.
- Lifecycle được chia làm 3 phần: **Mounting**, **Updating** and **Unmounting**

##### 3.2.2 Thứ tự của các lifecycle

- Mounting methods:

  - **constructor()**: hook này sẽ chạy khi component được khởi tạo. Chúng ta sẽ khởi tạo state hoặc là bind các function cần thiết trong này.
  - **componentWillMount()**: hook này sẽ chạy ngay sau constructor chạy xong (hook này đã bị deprecate khi react 16 release, nhưng trong phạm vi bài này mình vẫn sẽ giới thiệu tất cả cho mọi người biết vì biết đâu có dự án cũ vẫn sử dụng).
  - **render()**: hook này sẽ được gọi khá nhiều trong vòng đời của component, đc gọi lần đầu tiên sau khi componentWillMount được gọi, khi setState được gọi, khi componentWillReceiveProps....
  - **componentDidMount()**: hook này được gọi sau khi component đã render html ra DOM lần đầu tiên, và cũng chỉ được gọi đúng một lần trong vòng đời của component.

- Updating methods:

  - **componentWillReceiveProps()**: hook này sẽ chạy khi component nhận được props mới (hàm này đã deprecate trong phiên bản react 16). Vì rất nhiều lúc state sẽ phụ thuộc vào các props nhận được này nên ta có thể setState ở trong hook này.
  - **shouldComponentUpdate()**: hook này sẽ chạy ngay sau khi componentWillReceiveProps chạy xong. Nó sẽ trả về true hay false, tương ứng nó sẽ quyết định component có render lại hay không theo giá trị true hay false đó.
  - **componentWillUpdate()**: hook này sẽ được gọi khá nhiều trong vòng đời của component, đc gọi lần đầu tiên sau khi componentWillMount được gọi, khi setState được gọi, khi componentWillReceiveProps....
  - **render()**:
  - **componentDidUpdate()**: hook này được gọi sau khi component đã render html ra DOM lần đầu tiên, và cũng chỉ được gọi đúng một lần trong vòng đời của component.

- Unmounting methods:
  - **componentWillUnmount()**: hook này được gọi ngay trước khi component bị xoá khỏi DOM. Hook này là nơi bạn có thể thực hiện các thao tác xoá timer, cancel request, cancel event listener....
  - **componentDidCatch()**: Hook này sẽ được gọi khi component xảy ra bất cứ lỗi nào, ta có thể sử dụng setState trong này để catch các lỗi xảy ra. (hook này mới được thêm vào trong react 16).



##### Tài liệu tham khảo

https://stories.jotform.com/7-reasons-why-you-should-use-react-ad420c634247

https://www.c-sharpcorner.com/article/what-and-why-reactjs/

https://reactjs.org/docs/state-and-lifecycle.html
