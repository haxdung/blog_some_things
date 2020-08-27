+++
title = " Xử lý backgroud job với gem sidekiq"
description = "Giải pháp là extract các tác vụ tốn thời gian và thực thi chúng một cách không đồng bộ."
author = "DungHX"
date = "2020-08-26"
tags = ["ruby", "rails", "sidekiq", "backgroud job"]
categories = ["ruby", "rails"]
[[images]]
  src = "img/2020/tech/sidekiq.png"
  alt = "sidekiq image"
  stretch = "Vertical"
+++

### Bài Toán

Khi làm dự án chắc hẳn bạn đã gặp bài toán yêu cầu ứng dụng của bạn xử lý các yêu cầu từ web đến server một cách nhanh chóng. Một số yêu cầu thao tác dữ liệu từ cơ sở dữ liệu của bạn trên hàng triệu bản ghi mà vẫn phải nhanh mà không tốn thời gian, ví dụ:


Gửi email
Gửi thông báo
Thực hiện giao dịch
Tạo file pdf
Tương tác với các API của bên thứ 3 (craw dữ liệu...)
Thực hiện các task tốn nhiều thời gian ( update index trong database )

Thực hiện các tác vụ tốn thời gian trong quá trình xử lý request sẽ ảnh hưởng đến thời gian response.

### Giải pháp

Giải pháp là extract các tác vụ tốn thời gian và thực thi chúng một cách không đồng bộ

Một thành ngữ phổ biến là extract các tasks tiêu tốn thời gian thành các đơn vị công việc được gọi là job. Các job sẽ được đưa vào schedule để thực hiện lần lượt. Có rất nhiều dự án Ruby xử lý Background jobs với Sidekiq, DelayedJob, Resque ...

Chúng ta chọn Sidekiq vì Sidekiq cung cấp hiệu suất tối ưu, tính linh hoạt và hỗ trợ cho các mục đích của chúng ta. Để sử dụng Sidekiq vào dự án của bạn, bạn sẽ cần cài đặt thứ sau:

Database để lưu trữ jobs - Redis

Processor thực thi các jobs - Sidekiq

Bạn có thể tìm thấy hướng dẫn về cách cấu hình Sidekiq để hoạt động với Redis [tại đây](https://github.com/mperham/sidekiq/wiki/Using-Redis). Sau khi bạn [thiết lập worker](https://github.com/mperham/sidekiq/wiki/Getting-Started) (Background jobs), bạn có thể kết nối nó ở bất cứ nơi nào nó được yêu cầu và chọn thời điểm nó sẽ được thực thi.


###  1: Tổng quan về sidekiq

Sidekiq là framework để process background job cho phép scale up perform của Rails app gồm 3 phần chính: client, redis, server

**a, Client**
* Sidekiq client thực hiện chạy các Ruby process cho phép bạn tạo các background job.
```
MyWorker.perform_async(1, 2, 3)
Sidekiq::Client.push("class" => MyWorker, "args" => [1, 2, 3])
```

* Cả 2 method này đều tạo 1 hash để lưu các thông tin của job vừa được tạo, serializes hash thành chuỗi JSON và đưa vào queue của redis. Các tham số truyền vào bên trong worker đều phải là các kiểu dữ kiệu đơn giản được support bởi JSON như integer, float, string, boolean, array, hash.

**b, Redis**

Redis chứa tất cả data của các job được sidekiq push vào queue, lịch sử các job đã thực hiện cũng như các thông tin khác để hiển thị lên Web UI của Sidekiq

Cách mà sidekiq [connect](https://github.com/mperham/sidekiq/wiki/Using-Redis) tới redis

**c, Server**

Sidekiq server thực hiện đưa các job vào queue của Redis. Sidekiq server boots Rails vào trong các worker do đó bên trong các Worker có full các API của Rails (Active Record, ...)
Sidekiq server sẽ tạo các instance của worker và gọi instance method perfom khi worker được gọi.


### 2: Getting Started:
Thêm sidekiq vào Gemfile
```
# Gemfile
gem "sidekiq"
```
Chạy command bundle install để install gem
`bundle install`

Tạo worker để chạy các job bất đồng bộ

```
class HaloWorker
  include Sidekiq::Worker

  def perform(name, count)
    puts "HaloWorker is runing"

    count.times do
      puts "...Hello #{name}"
    end
  end
end
```

Hàm perform là instance method được gọi khi ta gọi class method perform_async, và hàm này  chỉ nhận các tham số đơn giản như string, integer, boolean được support bởi JSON.

Để start sidekiq ở môi trường development, ta phải start redis vad sidekiq

Command redis-server để chạy redis
```
redis-server
bundle exec sidekiq
```
Vào rails console để tạo 1 job bất đồng bộ

`HaloWorker.perform_async("Dung", 3)`


Ngoài ra Sidekiq còn cung cấp thêm 2 class method là **perform_in** và **perform_at** bên cạnh perform_async.

Với 2 class method này, job sẽ được lên schedule theo tham số truyền vào method chứ không thưc hiện ngay khi được lấy ra khỏi queue như method perform_async.

```
HaloWorker.perform_in(2.minutes, "Dung", 2)
HaloWorker.perform_at(2.minutes.from_now, "Dung", 3)
```

### 3: Config sidekiq

**The Sidekiq Configuration File**

Sidekiq config file là file YAML, mặc định là app/config/sidekiq.yml lưu các thông tin config cho Sidekiq server. Chỉ tạo file nếu cần custom các advance của sidekiq, nếu không sidekiq sẽ sử dụng các config mặc định.

```
# app/config/sidekiq.yml
:concurrency: 5
staging:
  :concurrency: 10
production:
  :concurrency: 20
:queues:
  - critical
  - default
  - low
```

**Queues:**

Theo mặc định, sidekiq sử dụng queue mặc định là default. Nếu muốn dử dụng nhiều queue, ta có thể khai báo tên và weight của chúng với -q option khi start sidekiq.

```
bundle exec sidekiq -q critical,2 -q default
```

Hoặc khai báo trong file app/config/sidekiq.yml

```
# app/config/sidekiq.yml
:queues:
  - [critical, 2]
  - default
```

Ngoài ra bạn cũng có thể khai báo các queue theo 1 thứ tự nhất định và các job được đưa vào các queue sẽ được thực hiện theo đúng thứ tự khai báo của các queue.

**Workers:**
Sidekiq cung cấp 1 số option sau cho các worker:
- Queue: sử dụng queue được đặt tên cho worker này, mặc định là default
- Retry: Enable re-execute job khi process thất bại, defaule là true và sẽ thực hiện retry tối đa 25 lần, ngoài ra bạn có thể chỉ định số lần tối đa một job được retry.
- Dead: việc một Job không thành công sẽ được chuyển đến hàng đợi nếu nó không retry được, mặc định: true
- Backtrace: Quy định có lưu error backtrace và hiển thị lên Web UI của side kiq hay không, nó có thể nhận giá trị boolean (true, false) hoặc integer (số dòng error backtrace được lưu lại), giá trị mặc định là false.
- Pool: sử dụng Redis connection pool để đẩy loại type của job đến một phân đoạn nhất định.

### 4: Scheduled Jobs
Sidekiq cho phép bạn config thời gian execute của job. Bạn có thể sử dụng class methof perform_in(interval_in_seconds, *args) và perform_at(timestamp, *args) để lên schedule cho jon thay vì sử dụng class methof perform_async(*args):
```
MyWorker.perform_in(3.hours, 'mike', 1)
MyWorker.perform_at(3.hours.from_now, 'mike', 1)
```

**Timezones**

Sidekiq's schedule gọi method to_f với tham số truyển vào perform_in và perform_at nên không bị phụ thuộc vào time zone.

**Checking for New Jobs**

Sidekiq kiểm tra các job đã được schedule theo interval khoảng 5s 1 lần
hoặc bạn có thể config lại như sau:

```
Sidekiq.configure_server do |config|
  config.average_scheduled_poll_interval = 15
end
```

Tài liệu tham khảo :

https://github.com/mperham/sidekiq/wiki

https://infinum.com/handbook/books/rails/development-practices/background-and-scheduled-jobs
