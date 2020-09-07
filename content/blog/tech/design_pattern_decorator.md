+++
title = "Giới thiệu về Design pattern Decorator"
description = "Decorator là một mẫu design pattern cho phép bạn gắn các method mới vào các đối tượng bằng cách đặt các đối tượng này bên trong các đối tượng bao bọc đặc biệt có chứa các method đó"
author = "DungHX"
date = "2020-08-29"
tags = ["rails", "ruby", "Design Patterns"]
categories = ["Design Patterns", "ruby"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/design_pattern_decorator.png"
  alt = "Giới thiệu về Design pattern Decorator"
  stretch = "horizontal"
+++

Decorator là một mẫu design pattern cho phép bạn gắn các method mới vào các đối tượng bằng cách đặt các đối tượng này bên trong các đối tượng bao bọc đặc biệt có chứa các method đó.

### Bài toán

Hãy tưởng tượng rằng bạn đang làm việc trên notification library cho phép các chương trình khác thông báo cho người dùng của họ về các sự kiện quan trọng.

Phiên bản ban đầu của notification library dựa trên Notifier class chỉ có một vài trường, hàm tạo và một phương thức send. Phương thức này có thể chấp nhận một số tin nhắn từ một khách hàng và gửi tin nhắn đến một danh sách các email đã được chuyển đến trình thông báo qua hàm tạo của nó. Một ứng dụng của bên thứ ba hoạt động như một ứng dụng khách có nghĩa vụ phải tạo và cấu hình đối tượng notificator một lần, sau đó sử dụng nó mỗi khi có các sự kiện quan trọng xảy ra.

![](https://images.viblo.asia/90e8a9ea-9bc8-4463-9d4e-2e730bf5ac0b.png)


Một lúc nào đó, bạn nhận ra rằng người dùng của notification library mong đợi nhiều hơn là chỉ thông báo qua email. Có thể họ muốn nhận được một tin nhắn SMS.Hay là những thông báo trên Facebook và hay cả thông báo của Slack.

![](https://images.viblo.asia/b8d1310d-4a77-4e68-8ae8-08c5164bc211.png)

Nó có thể khó đến mức nào? Bạn đã mở rộng Notifier class và đặt các phương thức notification bổ sung vào các lớp con mới. Bây giờ client có thể khởi tạo lớp notification mong muốn và sử dụng nó cho tất cả các notification tiếp theo.

Nhưng sau đó ai đó hỏi bạn rằng, Tại sao bạn không thể sử dụng nhiều loại notification cùng một lúc? Nếu mà có thông tin quan trọng nào đó khách hàng sẽ muốn được thông báo qua mọi kênh.

Bạn đã cố gắng giải quyết vấn đề đó bằng cách tạo các lớp con đặc biệt kết hợp một số phương thức notification trong một lớp. Tuy nhiên, ta có thể nhận thấy rằng cách tiếp cận này sẽ làm không phù hợp vì tốn nhiều dòng code hơn.

![](https://images.viblo.asia/972f740d-70e0-4fcf-8267-540faff13574.png)

Bây giờ ta phải tìm một số cách khác để cấu trúc các lớp notification

### Giải pháp

Mở rộng một class là giải pháp điều đầu tiên bạn nghĩ đến khi bạn cần thay đổi mothod của một đối tượng. Tuy nhiên, inheritance có một số cảnh báo nghiêm trọng mà bạn cần phải biết.

inheritance là tĩnh. Bạn không thể thay đổi method của một đối tượng hiện có trong thời gian chạy. Bạn chỉ có thể thay thế toàn bộ đối tượng bằng một đối tượng khác được tạo từ một lớp con khác.

Các lớp con có thể chỉ có một lớp cha. Trong hầu hết các ngôn ngữ, thừa kế không cho phép một lớp kế thừa các hành vi của nhiều lớp cùng một lúc.

Một trong những cách để khắc phục là sử dụng Composition thay vì Kế thừa. Với Composition, bạn có thể dễ dàng thay thế đối tượng với đối tượng khác, thay đổi method của vùng chứa khi chạy. Một đối tượng có thể sử dụng method của các lớp khác nhau, có các tham chiếu đến nhiều đối tượng.

![](https://images.viblo.asia/d3961856-57a1-4eed-96f8-e8ad58513055.png)


Wrapper là tên thay thế cho Decorator pattern thể hiện rõ ràng công dụng của Decorator pattern. Wrapper có các phương thức cho tất cả các yêu cầu mà nó nhận được.

Wrapper implements các interface như wrapped object. Đó là lý do tại sao từ quan điểm của khách hàng, các đối tượng này là giống hệt nhau. Điều này sẽ cho phép bạn cover một đối tượng trong nhiều wrappers, thêm method của tất cả các trình bao bọc vào nó.

![](https://images.viblo.asia/99bc791d-a0d5-4e0d-a885-893d15d894a1.png)

Mã khách hàng sẽ wrap một đối tượng notifier vào một tập hợp các decorators phù hợp với sở thích của khách hàng. Kết quả Các đối tượng sẽ được cấu trúc như một ngăn xếp.

![](https://images.viblo.asia/7ff374fc-dfb5-4b8a-9d2e-0a68c995e587.png)

Decorator cuối cùng trong ngăn xếp sẽ là đối tượng mà client thực sự làm việc với. Vì tất cả các decorator đều thực hiện cùng một interface với base notifier, phần còn lại của mã máy client sẽ không quan tâm liệu nó có hoạt động với đối tượng notificator hay không.

Ta có thể áp dụng cách tiếp cận tương tự cho các hành vi khác như định dạng tin nhắn hoặc soạn danh sách người nhận. client có thể decorate đối tượng với bất kỳ decorate tùy chỉnh nào, miễn là theo cùng một interface như interface khác.

Tài liệu tham khảo: https://refactoring.guru/design-patterns/decorator
https://thoughtbot.com/blog/evaluating-alternative-decorator-implementations-in
