+++
title = "Fat Model, bạn sẽ đặt logic ở đâu ?"
description = "Model là một thành phần cốt lõi của kiến trúc MVC, nơi tất cả logic được xử lý ở đó nên nó sẽ càng ngày càng lớn khi bạn xử lý logic quá nhiều ở đó, nhưng không phải lúc nào cũng để hết logic xử lý trong model"
author = "DungHX"
date = "2020-09-07"
tags = ["rails", "ruby", "fat model", "refactor"]
categories = ["rails", "ruby", "refactor"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/fat_model_ruby.png"
  alt = "Fat Model, bạn sẽ đặt logic ở đâu ?"
  stretch = "horizontal"
+++

Bài viết này sẽ tổng hợp một số kỹ thuật giúp xử lý khi mà model của bạn quá lớn hay quá phức tạp để xử lý nó. Khi model của bạn quá lớn nó sẽ gây ra khó khăn cho việc bảo trì hay khó khăn hơn trong việc xây dựng những hệ thống lớn.

Model là một thành phần cốt lõi của kiến trúc MVC, nơi tất cả logic được xử lý ở đó nên nó sẽ càng ngày càng lớn khi bạn xử lý logic quá nhiều ở đó, nhưng không phải lúc nào cũng để hết logic xử lý trong model.

Trong mô hình MVC mọi người thường cố gắng code sao cho Fat Models Thin Controllers Stupid view
=> Model sẽ chứa các đoạn code logic xử lý dữ liệu, controller lấy dữ liệu và view chỉ để hiển thị => Dễ thấy tất nhiên nên tránh fat controllers. Còn Fat models cũng cần tránh do nếu quá nhiều logic và truy vấn CSDL trong duy nhất một model cũng sẽ gây khó hiểu cho người đọc, khó có thể tái sử dụng code
- Tránh fat models và fat controller có thể giúp code của bạn dễ hiểu hơn, đồng thời giúp việc bảo trì và tái sử dụng code hiệu quả hơn
- Các kĩ thuật Value, Service, Form, View, Query, và Policy Objects là 6 trong 7 design pattern giúp loại bỏ fat models trong rails.
### 1. Value Object
- Là các object nhỏ, đơn giản, thường chỉ chứa các giá trị nhất định. Các value object thường không thể thay đổi và Date, URI, Pathname là các ví dụ của value object trong thư viện chuẩn của Ruby.
- Trong Rails, có thể sử dụng value object khi bạn có một hoặc nhiều attributes có các thao tác logic. Ví dụ một ứng dụng nhắn tin sẽ có một value object là PhoneNumber, một ứng dụng đánh giá sẽ có value object Rating như sau:
![](https://images.viblo.asia/014b8146-0d3e-4194-b92d-64ff70f949ae.png)
Trong model ConstantSnapshot sẽ sử dụng value object Rating như sau:
![](https://images.viblo.asia/50be8fe2-6cdd-4697-8b68-60b256b03e60.png)
### 2. Service object
Service object: được sử dụng trong trường hợp:
- Một thao tác phức tạp
- thao tác liên quan tới nhiều model khác nhau (một ứng dụng thương mại điện tử sẽ cần phải thao tác với các object CreditCard, Order, Customer,...)
- Một thao tác có nhiều bước
- Có nhiều cách để thực hiện thao tác (ví dụ có thể authenticate bằng password hoặc token)
- Ví dụ thay vì tạo một method authenticate trong class User ta có thể tạo một service object cho nó

![](https://images.viblo.asia/96ef0594-57f1-4231-a461-631c00a1d419.png)

### 3: Form Object
 Khi nhiều models được update thông qua một form submit, sử dụng Form Object để gộp các attributes của các model cần tác động sẽ gọn gàng hơn sử dụng nested_attributes (accepts_nested_attributes_for)
- Ví dụ khi người dùng đăng ký (create một record của user model) sẽ đồng thời tạo một record mới của Company model => Ta tạo một form object là class Signup

![](https://images.viblo.asia/12d5f98a-4f8d-4baa-9d41-e637e1c51516.png)

Virtus là gem cho phép định nghĩa các attribute trên một class, module, hoặc instance của class như một model. Form object Signup lúc này có thể coi như một ActiveRecord model nên controller sẽ như sau:

![](https://images.viblo.asia/716d6323-3cd6-4cff-8088-63b64c098942.png)

 Trong trường hợp xử lý các logic trong form phức tạp có thể sử dụng kết hợp Form Object với Service Object pattern để giảm thiểu fat models

### 4: Query Object
- Nếu trong model có nhiều scope phức tạp, controller nhiều truy vấn nối nhau, sử dụng các lệnh SQL thuần cồng kềnh => Cân nhắc sử dụng Query Object
- Mỗi Query Object là một class chịu trách nhiệm trả lại kết quả của một câu truy vấn dựa vào các quy tắc nghiệp vụ
- Ví dụ để lấy ra các sự kiện không có người tham dự có thể tạo một query object chứa truy vấn như sau:

![](https://images.viblo.asia/4115983b-2c82-40db-9409-6dab8a14a105.png)

### 5: View Object

Nếu một xử lý logic chỉ để phục vụ cho vấn đề hiển thị thì không nên viết nó trong model => Ngoài cách viết helper ra thì có thể tạo một View Object. Tất cả logic ở trong View thì gộp thành một object. Ví dụ:

![](https://images.viblo.asia/03aae69d-110b-4950-80b1-1879ec8f4f07.png)

Trong controller sẽ gọi lại như sau:
![](https://images.viblo.asia/0ef9f11b-5206-4a6b-80b8-8c9db51d46c2.png)

Ở view sẽ gọi lại như sau
![](https://images.viblo.asia/38a27a1e-19dd-472c-9004-d1f6696f02fd.png)

### 6: Policy Object
 Policy Object gần giống với Service Object. Ví dụ chúng ta có thể check quyền của người dùng thao tác với dữ liệu sử dụng Policy Object. Các Policy Object được dùng chủ yếu trong việc phân quyền

![](https://images.viblo.asia/e6550b4c-3c1f-46a0-83e4-c1090ac1c778.png)

Policy Object này sẽ thực hiện việc kiểm tra user có active hay không nếu user có địa chỉ email confirm và lần cuối đăng nhập trong khoảng 2 tuần gần đây.


*Nguồn tham khảo:*

https://medium.com/@jaysadiq/refactoring-a-fat-rails-model-dc3cfda64d22
 https://www.justinweiss.com/articles/when-is-an-activerecord-model-too-fat/
