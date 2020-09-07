+++
title = "Khi nào nên dùng Render, Redirect_to, vì sao ?"
description = "Với những người mới bắt đầu tìm hiểu về ROR, phương thức render và redirect_to có thể khó phân biệt được. Hai phương thức này đều xuất hiện ở cuối các action của controller, tạo HTTP response để trả về và sau cùng đều hiển thị view mới trên web browser"
author = "DungHX"
date = "2020-09-01"
tags = ["rails", "ruby", "render", "redirect_to"]
categories = ["rails", "ruby"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/render_or_redirect_to.png"
  alt = "Khi nào nên dùng Render, Redirect_to, vì sao ?"
  stretch = "horizontal"
+++

Trong Rails luồng xử lý controller/view thông thường sẽ là hiển thị một template view tương ứng với action của controller xử lý. Rails có cung cấp 2 method là render trong controller nếu chúng ta muốn có respond ở request hiện tại, và redirect_to nếu muốn tạo request mới. Với những người mới bắt đầu tìm hiểu về ROR, phương thức render và redirect_to có thể khó phân biệt được. Hai phương thức này đều xuất hiện ở cuối các action của controller, tạo HTTP response để trả về và sau cùng đều hiển thị view mới trên web browser. Sau đây mình sẽ chỉ ra sự khác nhau cho từng trường hợp.

### 1: Render
Gọi render sẽ tạo một response đầy đủ trả về cho browser. Nếu không chỉ rõ render trong controller action, Rails sẽ tự động tìm kiếm và render template tương ứng dựa vào tên controller action.

![](https://images.viblo.asia/8942ff02-abbd-461e-8f04-55241bed5db6.png)

*  Mặc định render sẽ cố gắng hiển thị view template của action mà nó được gọi ở trong tương ứng. Như ví dụ trên nếu không chỉ rõ render :edit thì render sẽ cố gắng render file update.html.erb, còn nếu chỉ rõ tên action là edit thì sẽ render ra file edit.html.erb (lưu ý là render :edit sẽ chỉ hiển thị ra view template edit chứ không thực thi đoạn code logic trong action edit của controller)
*  Có thể sử dụng render để render view template của các action thuộc về các controller khác nhau

![](https://images.viblo.asia/5f217869-ec4e-4eee-bd4a-ae01b6e7252e.png)
                     => render ra view template của ErrorsController#show

* Ngoài ra có thể sử dụng render để hiển thị trực tiếp dữ liệu mà không cần có một view template

### 2: Redirect_to

* Sử dụng redirect_to khi muốn tạo request mới. Nhưng tại sao lại cần tạo request?
* Ví dụ nếu người dùng tạo một bài post và lưu lại chúng ta sẽ mong muốn hiển thị cho người dùng thấy bài post họ vừa tạo. Người dùng đã tạo một POST request gửi dữ liệu lên server. Việc hiển thị bài post vừa tạo hoàn toàn có thể sử dụng render trong cùng POST request đó.
*  Nhưng nếu người dùng refresh lại trang thì sao? Hoặc click vào một đường dẫn rồi back trở lại trang gửi dữ liệu? Họ sẽ thấy một pop-up hỏi có muốn submit lại dữ liệu lên server không => gây khó hiểu cho người dùng

![](https://images.viblo.asia/fb9165ef-f7bc-40bb-b397-e52c0d04e19d.png)

*  Do vậy sau khi lưu dữ liệu thành công tốt nhất nên tạo một request mới để chuyển hướng người dùng tới một trang khác. Do vậy trong trường hợp người dùng muốn refresh hay back lại trang, họ sẽ không phải nhận popup như trong trường hợp nếu sử dụng render

Lưu ý là redirect_to không làm action dừng thực thi như method return trong Ruby
Giả sử có một action delete như sau:

![](https://images.viblo.asia/7a4f65f7-e901-495b-820d-3d33ac2782e7.png)

Có người muốn bổ sung thêm logic như sau: nếu người dùng không là admin thì sẽ không thực hiện được action delete mà thay vào đó sẽ chuyển tới trang login
![](https://images.viblo.asia/08b21656-0c4a-41e5-bf46-47da1eb18670.png)

Trong trường hợp này tất nhiên admin vẫn xóa post thành công do điều kiện unless là true => câu redirect_to đầu tiên được bỏ qua

Với người dùng không phải admin, unless sẽ trả về false, do đó lệnh redirect_to đầu tiên sẽ chạy, một response chuyển trang sẽ được thiết lập và các câu lệnh sau vẫn được thực thi => bài post bị xóa

Lệnh redirect_to tiếp theo chạy, nhưng do đã có một response chuyển trang khác đã được thiết lập (ở lệnh redirect_to đầu tiên) nên sẽ tạo ra một exception là DoubleRender

Lý do là lệnh redirect_to không làm đoạn code dừng thực thi mà nó chỉ thiết lập thêm thông tin ở response trả về.



Tài liệu tham khảo:

https://tosbourn.com/difference-between-redirect-render-rails/ http://guides.rubyonrails.org/layouts_and_rendering.html#using-render
