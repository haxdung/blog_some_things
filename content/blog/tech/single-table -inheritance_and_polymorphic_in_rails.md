+++
title = "Single-Table Inheritance và polymorphic trong Rails"
description = "Nếu bạn đã từng tạo một ứng dụng có nhiều models, thì bạn đã phải suy nghĩ về việc sử dụng loại mối quan hệ nào giữa các models đó"
author = "DungHX"
date = "2020-09-02"
tags = ["STI", "ruby", "polymorphic", "rails"]
categories = ["rails", "ruby"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/sti_and_polymorphic.png"
  alt = "Single-Table Inheritance và polymorphic trong Rails"
  stretch = "horizontal"
+++

Nếu bạn đã từng tạo một ứng dụng có nhiều models, thì bạn đã phải suy nghĩ về việc sử dụng loại mối quan hệ nào giữa các models đó.

Khi độ phức tạp của ứng dụng tăng lên, có thể khó quyết định mối quan hệ nào nên tồn tại giữa các models.

Một tình huống thường xuyên xảy ra là khi một số model của bạn cần có quyền truy cập vào chức năng của model thứ ba. Hai phương thức mà Rails cho chúng ta để giải quyết bài tóan này là kế thừa bảng đơn (STI)  và  polymorphic.

In Single-Table Inheritance (STI) nhiều lớp con kế thừa từ một superclass với tất cả dữ liệu trong cùng một bảng trong cơ sở dữ liệu. Superclass có một cột "type" để xác định lớp con của một đối tượng.

Trong một polymorphic, một model, thuộc về một số model khác sử dụng một liên kết duy nhất, Mỗi mô hình, bao gồm mô hình polymorphic, có bảng riêng trong cơ sở dữ liệu.

Vậy đối với từng phương pháp thì khi nào ta sử dụng chúng:

### Single-Table Inheritance

Ví dụ chúng ta tạo có một app liệt kê các loại xe khác nhau được bán tại cửa hàng, cửa hàng này sẽ bán cars, motorcycles, and bicycles. Đối với mỗi chiếc xe, cửa hàng muốn theo dõi giá cả, màu sắc và liệu chiếc xe đã được mua. Tình huống này kỹ thuật STI sẽ được áp dụng, bởi vì chúng tôi đang sử dụng cùng một dữ liệu cho mỗi class.

Chúng ta có thể tạo ra một chiếc siêu xe với các thuộc tính về màu sắc, giá cả và được mua. Mỗi class con kế thừa từ class Vehicle và tất cả đều có thể có được các thuộc tính.

Tạo bảng CreateVehicles như sau:
```
class CreateVehicles < ActiveRecord::Migration[5.1]
    def change
        create_table :vehicles do |t|
            t.string :type, null: false
            t.string :color
            t.integer :price
            t.boolean :purchased, default: false                                                   end
    end
end
```
Chúng ta tạo một cột type để cho superclass, điều này cho Rails biết rằng chúng ta đang sử dụng STI và muốn tất cả dữ liệu cho Vehicles và các lớp con của nó nằm trong cùng một bảng trong cơ sở dữ liệu.

model classes  sẽ như sau:
```
class Vehicle < ApplicationRecordend
class Bicycle < Vehicleend
class Motorcycle < Vehicleend
class Car < Vehicleend
```

Phương pháp này rất tốt vì mọi phương thức hoặc validations trong lớp Vehicles được chia sẻ với mỗi lớp con của nó. Chúng ta có thể thêm các phương thức duy nhất vào bất kỳ lớp con nào nếu cần, chúng độc lập với nhau.

Ngoài ra, vì chúng ta biết rằng các lớp con chia sẻ cùng một trường dữ liệu, chúng ta có thể gọi các đối tượng từ các lớp khác nhau.
```
mustang = Car.new(price: 50000, color: red)harley = Motorcycle.new(price: 30000, color: black)
mustang.price=> 50000
harley.price=> 30000
```
### Polymorphic Associations

Với Polymorphic, một model có thể thuộc về nhiều models với một liên kết duy nhất.

Polymorphic được áp dụng khi một số model không có mối quan hệ hoặc chia sẻ dữ liệu với nhau, nhưng có mối quan hệ với lớp polymorphic.

Lấy ví dụ, chúng ta có một trang web truyền thông xã hội như Facebook. Trên Facebook, cả cá nhân và nhóm đều có thể chia sẻ bài đăng. Các cá nhân và nhóm không liên quan (ngoài cả hai là một loại người dùng) và do đó họ có dữ liệu khác nhau. Một nhóm có thể có các trường như Member_count và group_type áp dụng cho một cá nhân và ngược lại)

Nếu không sử dụng polymorphic, ta có:
```
class Post  belongs_to :person  belongs to :groupend
class Person  has_many :postsend
class Group  has_many :postsend
```
Thông thường, để tìm thông tin của user, ta dựa vào cột đó là Foreign_key. Foreign_key là id được sử dụng để tìm đối tượng liên quan trong bảng. Tuy nhiên, bảng Posts của ta sẽ có hai khóa ngoại là: group_id và person_id. Đây là vấn đề

Khi cố gắng tìm user của một bài đăng nào, ta sẽ phải đưa ra một điều kiện để kiểm tra cả hai cột để tìm chính xác Foreign_key, thay vì dựa vào một cột. Điều gì xảy ra nếu chúng ta gặp phải tình huống cả hai cột có giá trị?

Polymorphic đã giải quyết vấn đề này bằng cách condensing các chức năng này thành một association.

```
class Post  belongs_to :postable, polymorphic: trueend
class Person  has_many :posts, as :postableend
class Group  has_many :posts, as :postableend
```
Quy ước Rails để đặt tên cho Polymorphic sử dụng tính năng có thể sử dụng được với tên lớp (có thể gửi được cho lớp Post). Điều này làm cho nó rõ ràng trong các mối quan hệ của bạn mà lớp là Polymorphic. Nhưng bạn có thể sử dụng bất kỳ tên nào cho Polymorphic. Để sử dụng với cơ sở dữ liệu, ta sử dụng một liên kết Polymorphic, ta sử dụng các cột điều kiện kiểu kiểu YouTube và kiểu Id cho lớp Polymorphic. Cột postable_type ghi lại mô hình mà post belongs to, trong khi cột postable_id theo dõi id của đối tượng sở hữu.
```
haley = Person.first=> returns Person object with name: "Haley"
article = haley.posts.firstarticle.postable_type=> "Person"
article.postable_id=> 1 # The object that owns this has an id of 1 (in this case a      Person)
new_post = haley.posts.new()# Automatically fills in postable_type and postable_id using haley object
```
Polymorphic là sự kết hợp của hai hoặc nhiều association belongs to. Do đó, bạn có thể sử dụng giống như khi bạn sử dụng hai models có liên kết belongs to.
```
haley.posts# returns ActiveRecord array of posts
haley.posts.first.content=> "The content from my first post was a string..."
```
### Làm thế nào để biết sử dụng giải pháp nào:

STI và Polymorphic hay nhầm lẫn trong các trường hợp sử dụng. Mặc dù không phải là giải pháp duy nhất cho association model relationship, nhưng cả hai đều có một số lợi thế rõ ràng.

Cả hai ví dụ Vehicles và Postable đều có thể được thực hiện bằng một trong hai phương pháp. Tuy nhiên, có một vài lý do cho thấy rõ phương pháp nào là tốt nhất trong từng tình huống.
Chúng ta phai dựa vào Database structure, Shared data or state, Future concerns, Data integrity.

Tài Liệu tham khảo: https://www.freecodecamp.org/news/single-table-inheritance-vs-polymorphic-associations-in-rails-af3a07a204f2/
