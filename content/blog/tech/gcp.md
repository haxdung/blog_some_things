+++
title = "Tìm hiểu về Cloud Datastore trong Google Cloud Platform (GCP)"
description = "Google Cloud Platform là nền tảng điện toán đám mây do Google cung cấp, với các dịch vụ máy ảo, lưu trữ, phân tích dữ liệu, cùng nhiều công nghệ tiên tiến khác."
author = "DungHX"
date = "2020-08-25"
tags = ["ruby", "rails", "GCP", "Datastore"]
categories = ["ruby", "rails"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/gcp.png"
  alt = "GCP image"
  stretch = "horizontal"
+++

Mình đã trở lại rồi đây, dạo gần đây dự án mình đang làm có sử dụng dịch vụ Storage của Google cloud và mình cũng được tìm hiểu và làm việc với nó, bài hôm nay mình sẽ giới thiệu các bạn qua dịch vụ Cloud Datastore và các thao tác cơ bản với dữ liệu ở trong này mà mình hay dùng trong dự án.

Let's go =))
### 1: **Giới thiệu về Google Cloud Platform**

– Google Cloud Platform là nền tảng điện toán đám mây do Google cung cấp, với các dịch vụ máy ảo, lưu trữ, phân tích dữ liệu, cùng nhiều công nghệ tiên tiến khác.

– Google Cloud được hỗ trợ, củng cố và đổi mới bởi một cơ sở hạ tầng các sản phẩm của Google.

– Google Cloud đã mở rộng bảy sản phẩm, mỗi sản phẩm có hơn một tỷ người dùng, mỗi ngày Google xử lý 1,4 petabyte thông tin.

– Google Cloud có khả năng xây dựng, tổ chức và vận hành một mạng lưới lớn các máy chủ và cáp quang.

– Mạng lưới kết nối vật lý của nó là hàng ngàn dặm cáp quang, và hàng ngàn máy chủ tập hợp lại.

Các sản phẩm của Google Cloud Platform thì có rất nhiểu:
![](https://images.viblo.asia/9c85782f-92c3-4673-93cc-ef70d1605188.png)

trong đó phải kể đến như dịch vụ Storage (GCS)


### 2: **Google Cloud Storage là gì ?**

Google Cloud Storage (GCS) là một Object Storage. Đây là một kho lưu trữ dữ liệu với chức năng quản lý đơn giản, và có khả năng mở rộng gần như vô hạn.

Khi bắt đầu làm quen với GCP bạn sẽ nhớ 3 kiến thức cơ bản sau:
* GCP Project :  Đơn vị để tính phí thanh toán dịch vụ
* Bucket : Xem như là một Container để lưu trữ file
* Object : File được lưu trữ

Đầu tiên chúng ta phải khởi tạo bucket để lưu trữ dữ liệu. Sau đó, ta sẽ chỉ định tên của bucket, nơi lưu trữ dữ liệu và Storage class cho bucket đó. Ta cũng có thể thay đổi Storage class theo từng Object

Trong GCS, có tất cả 4 Storage Class với khả năng sử dụng, thời gian lưu trữ tối thiểu, phí sử dụng khác nhau tuỳ theo từng class. 4 Storage class đấy là : Multi-Regional, Regional, Nearline, Coldline.

Ngoài ra, dưới đây là những chức năng mà tất cả các class trên đều hỗ trợ:
* Sử dụng Tools giống nhau hoặc API để truy cập dữ liệu
* (XML API, JSON API, gsutil, GCP Console, Client Library)
* Bảo mật dữ liệu bằng chứng thực OAuth…
* Đảm bảo độ bền dữ liệu lên đến 99.9999999% (9 số 9)
* Latency thấp
* Mã hoá dữ liệu



 Performance:

 Khả năng Input / output (I/O) ban đầu của bucket vào khoảng 1000 request Input và 5000 request output mỗi giây. Như vậy, đối với một Object có dung lượng của 1MB, thì 1 tháng trung bình có thể thực hiện 2.5 PB Input và 13PB output.  Trường hợp số request vào bucket chỉ định tăng đột biến, thì GCS sẽ tự động phân bố tài nguyên vào các server khác để tăng dung lượng I/O của bucket đó.



 Và trong dịch vụ  Storage CỦA GCP thì chúng ta sẽ đi tìm hiểu về Cloud Datastore.

 **Cloud Datastore**

Cloud Datastore là một Full managed NoSQL Database với khả năng mở rộng cực lớn, được sử dụng trong Back-end của các ứng dụng. Datastore sẽ tự động xử lý Sharding (phân chia dữ liệu rôi nạp vào các server khác nhau) và replication rồi tự động tiến hành scale cho phù hợp với tài nguyên của ứng dụng đó. Một chức năng vô cùng hữu dụng đối với databse.

 Google Cloud Datastore cung cấp cho bạn một cơ sở dữ liệu định hướng tài liệu có tính đàn hồi, có sẵn rất cao. Nó được quản lý đầy đủ vì vậy bạn không cần triển khai, cập nhật, cấu hình hoặc quản lý giải pháp cơ sở dữ liệu của mình. Trong datastore dữ liệu sẽ được chuyển theo kiểu byte array, rồi được lưu vào Bigtable. Các truy vấn (query) của Datastore sẽ lấy kết quả bằng cách sử dụng 1 hoặc nhiều index. Index được định nghĩa từ danh sách xếp theo thứ tự xử lý các thuộc tính khác nhau theo từng entity (thực thể) đặc thù.

Ưu điểm của Datastore:
*  Cơ sở dữ liệu NoSQL có thể mở rộng lên đến hàng tỷ dòng.
*  Full quyền quản lý dịch vụ.
*  Tự động xử lý Sharding và trùng lặp.
*  Hỗ trợ cho các giao thức ACID, truy vấn như câu lệnh SQL.
*  Tốc độ nhanh và khả năng mở rộng cao
*  Truy vấn từ mọi nơi thông qua RESTful API.

Datastore thích hợp với những ứng dụng có cấu trúc dữ liệu với khả năng mở rộng cực lớn.

Ngoài ra, Datastore còn được sử dụng làm databse cho GAE (Google App Engine). Nếu kết hợp với GAE, bạn sẽ dễ dàng tạo được những ứng dụng có chức năng auto scale một cách đơn giản.

Datastore sẽ thích hợp sử dụng với:

* Lưu trữ Profile user
* Product Catalogue
* Game data

Ngoài ra thì Google cho bạn hạn mức sử dụng miễn phí Cloud Datastore trong 1 ngày . Bạn chỉ phải trả phí cho phần vượt quá mức free trên.


### 3: Tìm hiểu về Cloud Firestore

Firestore trong  Datastore mode là một cơ sở dữ liệu NoQuery được xây dựng để tự động mở rộng, hiệu năng cao và dễ dàng phát triển ứng dụng. Firestore là phiên bản mới nhất của Datastore và giới thiệu một số cải tiến so với Datastore.

**Entities, Properties, and Keys**

Các đối tượng dữ liệu trong firestore trong datastore mode được gọi là entities. Một entities có một hoặc nhiều properties được đặt tên, mỗi properties có thể có một hoặc nhiều giá trị. Các entities cùng loại không cần phải có cùng properties và các giá trị của một entities cho một property nhất định không phải tất cả đều phải cùng loại dữ liệu

Datastore mode hỗ trợ nhiều loại dữ liệu cho các giá trị của property.
* Integers
* Floating-point numbers
* Strings
* Dates
* Binary data


Mỗi entity trong datastore có một key xác định duy nhất nó. Key này bao gồm các thành phần sau:

* namespace của entity, cho phép đa nhiệm
* kind của entity, dùng để truy vấn
* Một định danh cho các entity riêng lẻ, có thể là  key name string hay integer numeric ID
* ancestor path của entity trong database


Một ứng dụng có thể tìm và get một entity riêng lẻ từ database bằng key của entity hoặc nó có thể truy xuất một hoặc nhiều entity bằng cách đưa ra một truy vấn dựa trên các key hoặc giá trị property của entity.

**Các thao tác với entities**

Các ứng dụng có thể sử dụng API Firestore trong  Datastore để tạo, truy xuất, cập nhật và xóa các entities

 1: Creating entity

Bạn tạo một entity mới bằng cách khởi tạo nó và thiết lập các properties của nó

```
task = datastore.entity "Task", "sampleTask" do |t|
  t["category"] = "Personal"
  t["done"] = false
  t["priority"] = 4
  t["description"] = "Learn Cloud Datastore"
end
datastore.save task
```

Bạn có thể lưu entities vào database bằng cách sử dụng save

2: get data entity

Để lấy một entity từ database, hãy sử dụng key của nó để tìm kiếm

```
task_key = datastore.key "Task", "sampleTask"
task = datastore.find task_key
```

3: update entity

Để cập nhật một entity đã tồn tại, sửa đổi các thuộc tính của thực thể được truy xuất trước đó và lưu trữ nó bằng key của entity đó:
```
datastore.transaction do |_tx|
  task = datastore.find "Task", "sampleTask"
  task["priority"] = 5
  datastore.save task
end
```

4: Delete entity

cũng như update bạn cần phải tìm theo key của entity, bạn có thể xóa được :

```
task_key = datastore.key "Task", "sampleTask"
datastore.delete task_key
```

5: Ngoài ra chúng cũng có thể thao tác với nhiều entity cùng lúc, các thao tác với từng entity sẽ được thực hiện song song.

ví dụ việc tạo 2 entity cùng nhau như sau:
```
task1 = datastore.entity "Task" do |t|
  t["category"] = "Personal"
  t["done"] = false
  t["priority"] = 4
  t["description"] = "Learn Cloud Datastore"
end

task2 = datastore.entity "Task" do |t|
  t["category"] = "Personal"
  t["done"] = false
  t["priority"] = 5
  t["description"] = "Integrate Cloud Datastore"
end

tasks = datastore.save task1, task2
task_key1 = tasks[0].key
task_key2 = tasks[1].key
```

6: Datastore còn hỗ trợ việc tạo entity có quan hệ parent với nhau:

```
task_key = datastore.key [["TaskList", "default"], ["Task", "sampleTask"]]
```

Trên là những thao tác với datastore mình hay gặp ở dự án nhưng bên cạnh đó datasore còn có transaction hay các thứ hay ho khác, tới đây bài cũng đã dài nên mình sẽ giới thiệu với các bạn phần tiếp theo nhé :)

Tài liệu tham khảo :
https://cloud.google.com/datastore/docs/concepts/entities#datastore-datastore-basic-entity-ruby
