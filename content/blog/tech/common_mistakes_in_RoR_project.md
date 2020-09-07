+++
title = "Những lỗi hay gặp trong dự án RoR"
description = "Những lỗi thường gặp nhất trên các website sử dụng Ruby on Rails và các cách khắc phục nó ?"
author = "DungHX"
date = "2020-08-10"
tags = ["rails", "ruby"]
categories = ["rails", "ruby", "errors"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/error.png"
  alt = "Những lỗi hay gặp trong dự án RoR"
  stretch = "horizontal"
+++

Rails là một framework mã nguồn mở được sử dụng rộng rãi trên thế giới, được xây dựng dựa trên ngôn ngữ lập trình Ruby với tiêu chí làm đơn giản hóa quá trình phát triển ứng dụng web. Rails rất dễ sử dụng, nhưng cũng dễ mắc lỗi.  Bài viết này sẽ thống kê những lỗi thường gặp nhất trên các website sử dụng Ruby on Rails và các cách khắc phục.

Bài viết tập trung vào các lỗi có khả năng ảnh hưởng đến bạn và người dùng của bạn. Để làm điều này, các lỗi đã được xếp hạng theo số lượng dự án gặp phải trên các công ty khác nhau.

![](https://images.viblo.asia/7a4fe55c-79cc-4c44-a5c7-5163757b198f.png)


### 1. ActionController::RoutingError
Đây là lỗi được raise phổ biến nhất, có lẽ cũng không khó hiểu lắm. ActionControll :: RoutingError có nghĩa là người dùng đã yêu cầu một URL không tồn tại trong ứng dụng của bạn. Rails sẽ ghi lại điều này và nó sẽ giống như một lỗi, nhưng đối với hầu hết các phần thì đó không phải là lỗi của ứng dụng của bạn. Vấn đề có thể do attacker dùng bot để dò lỗi trên hệ thống của chúng ta hoặc người dùng gõ vào url đường dẫn không chính xác
```
ActionController::RoutingError (No route matches [GET] "/wp-admin"):
```

Bạn có thể nhận ra rằng, ActionController::RoutingError đó là do ứng dụng của bạn chứ không phải do người dùng gõ sai: nếu bạn deploy ứng dụng của mình lên Heroku hoặc bất kỳ nền tảng nào không cho phép bạn cung cấp các tệp tĩnh, sau đó bạn có thể thấy rằng CSS và JavaScript của mình không load được. Nếu điều đó xảy ra, log sẽ báo như tn :

```
ActionController::RoutingError (No route matches [GET] "/assets/application-eff78fd93759795a7be3aa21209b0bd2.css"):
```
Để sửa lỗi trên chúng ta thêm config vào file production.rb

```
Rails.application.configure do
  # other config
  config.public_file_server.enabled = true
end
```
### 2. NoMethodError: undefined method '[]' for nil:NilClass
Lỗi này thường xảy ra khi chúng ta muốn gọi 1 key trong hash hoặc 1 thuôc tính của đối tượng, nhưng đối tượng lấy ra lại đang trả về nil. Điều này có thể xảy ra khi bạn phân tích và trích xuất dữ liệu từ API JSON hoặc tệp CSV hoặc chỉ nhận dữ liệu từ các tham số lồng nhau trong hành động của controller. Giả sử chung ta có params như sau:
```
{ user: { address: { street: '123 Fake Street', town: 'Faketon', postcode: '12345' } } }
```

Khi ta lấy ra tên đường phố bằng cách params[:user][:address][:street], nhưng rất tiếc người dùng không nhập vào address nên user[:address] đã trả về nil.

Bạn có thể thực hiện kiểm tra không cho từng tham số và quay lại bằng cách sử dụng toán tử &&.
```
street = params[:user] && params[:user][:address] && params[:user][:address][:street]
```

Ta có một cách tốt hơn để truy cập các phần tử lồng nhau trong hashes, mảng và các đối tượng sự kiện như sau. Với ruby 2.2.3 trở lên, hàm dig cho phép ta đào sâu vào hash và lấy ra giá trị, nếu không có sẽ trả về nil thay vì raise ra lỗi. Để lấy ra tên đường phố ta làm như sau
```
street = params.dig(:user, :address, :street)
```
Bạn sẽ không nhận được bất kỳ lỗi nào từ việc sử dụng dig, mặc dù bạn cần lưu ý rằng street vẫn có thể nil.

Bên cạnh đó, nếu bạn cũng đang lấy qua các đối tượng lồng nhau bằng cách sử dụng ký hiệu chấm, bạn cũng có thể thực hiện việc này một cách an toàn trong Ruby 2.3, bằng cách sử dụng navigation operator an toàn. Vì vậy, thay vì gọi

```
street = user.address.street
```
và nhận NoMethodError: undefined method street for nil:NilClass bạn có thể gọi:
```
street = user&.address&.street
```
Nếu đang sử dụng các phiên bản ruby cũ hơn, chung ta có thể thêm gem ruby_dig để dùng hàm dig.

### 3. ActionController::InvalidAuthenticityToken

Lỗi thứ 3 mà mọi người hay mắc phải liên quan đến vấn đề bảo mật trong ứng dung của bạn. ActionController::InvalidAuthenticityToken sẽ được báo khi request POST, PUT, PATCH hoặc DELETE bị thiếu hoặc có mã thông báo CSRF (Cross Site Request giả mạo) không chính xác. CSRF là một lỗ hổng tiềm năng trong các ứng dụng web, trong đó một trang web độc hại đưa request cho ứng dụng của bạn thay vì một người dùng không biết. Nếu người dùng đăng nhập vào session, cookie của họ sẽ được gửi cùng với request và kẻ tấn công có thể thực thi các lệnh như người dùng.
Rails giảm nhẹ các cuộc tấn công CSRF bằng cách bao gồm mã thông báo an toàn trong tất cả các method  và xác minh bởi trang web, nhưng không thể được bên thứ ba biết đến. Điều này được thực hiện bởi  ApplicationContoder
```
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end
```
Nếu nhận được lỗi như trên nhiều khả năng user đã bị hacker tấn công lấy được phiên đăng nhập, nhưng rails đã chặn được các request giả mạo đó do token không khớp. Tuy nhiên có nhiều lý do khác khiến bạn có thể vô tình nhận được lỗi này.
* Ajax
* Ví dụ chúng ta gửi 1 request ajax từ client, chúng ta phải gửi kèm token theo request, nếu bạn đang gửi ajax bằng jquery, Rails đã xử lí phần này cho chúng ta rồi. Trong trường hợp bạn muốn tự làm, có thể làm như sau:

Thêm thẻ csrf token vào application layout
```
<%= csrf_meta_tags %>
```
Khi gửi request ajax, lấy token ra và gửi kèm theo header
```
const csrfToken = document.querySelector('[name="csrf-token"]').getAttribute('content');
fetch('/posts', {
  method: 'POST',
  body: JSON.stringify(body),
  headers: {
    'Content-Type': 'application/json'
    'X-CSRF-Token': csrfToken
  }
).then(function(response) {
  // handle response
});
```
* Webhooks/API Đôi khi chúng ta muốn gọi 1 request từ một hệ thống khác, lúc này chúng ta có thể tắt tính năng csrf token, tuy nhiên cần có cơ chế bảo mật và đưa các endpoint vào danh sách để đảm bảo request của chúng ta đến từ các nguồn tin cậy

```
class ApiController < ApplicationController
 skip_before_action :verify_authenticity_token
end
```
### 4. Net::ReadTimeout
Lỗi này xảy ra khi ruby tốn nhiều thời gian để đọc dữ liệu từ socket hơn thời gian timeout.(mặc định là 60s). Có nhiều nguyên nhân dẫn tới lỗi này, tùy vào server và thiết kế hệ thống của bạn, chúng ta có thể lựa chọn cách nâng thời gian time out nếu server của bạn cần nhiều thời gian trả về dữ liệu hơn.

### 5: ActiveRecord::RecordNotUnique: PG::UniqueViolation

Thông báo lỗi này xảy ra ở cơ sở dữ liệu PostgreQuery, ActiveRecord cho MySQL và SQLite sẽ đưa ra các lỗi tương tự. Vấn đề ở đây là một bảng cơ sở dữ liệu trong ứng dụng của bạn có index duy nhất trên một hoặc nhiều trường và một transaction đã được gửi đến cơ sở dữ liệu vi phạm index đó. Giả sự chúng ta tạo bảng User, và đánh index unique cho trường email

```
class CreateUsers < ActiveRecord::Migration[5.1]
 def change
   create_table :users do |t|
     t.string :email
     t.timestamps
   end
   add_index :users, :email, unique: true
 end
end
```
Để tránh lỗi này ở tầng DB, chúng ta có thể viết validate ở model user thông báo cho người dùng email trùng với đã được đăng kí.
```
class User < ApplicationRecord
  validates_uniqueness_of :email
end
```

Ngoài ra trong 1 số trường hợp, lỗi này xảy ra khi người dùng double click form dẫn đến việc submit và tạo nhiều bản ghi giống nhau cùng 1 lúc, do request xảy ra đồng thời nên có thể pass qua validate và chỉ bị chặn khi insert dữ liệu vào DB. Để ngăn điều này chúng ta nên disable nút submit khi người dùng submit form, ví dụ:

```
def self.set_flag( user_id, flag )
  # Making sure we only retry 2 times
  tries ||= 2
  flag = UserResourceFlag.where( :user_id => user_id , :flag => flag).first_or_create!
rescue ActiveRecord::RecordNotUnique => e
  Rollbar.error(e)
  retry unless (tries -= 1).zero?
end
```
Nguồn tham khảo: https://dzone.com/articles/top-10-errors-from-1000-ruby-on-rails-projects-and
