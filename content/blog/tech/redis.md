+++
title = "Cách sử dụng cơ sở dữ liệu Redis trong Ruby"
description = "Làm thế nào để giải quyết được bài toán khi mà dự án của bạn có hàng trăm nghìn bản ghi, số lượng người truy cập vào cùng lúc lên đến cả trăm nghìn người"
author = "DungHX"
date = "2020-08-27"
tags = ["ruby", "rails", "redis", "cache"]
categories = ["ruby", "rails"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/redis.png"
  alt = "redis image"
  stretch = "horizontal"
+++

Làm thế nào để giải quyết được bài toán khi mà dự án của bạn có hàng trăm nghìn bản ghi, số lượng người truy cập vào cùng lúc lên đến cả trăm nghìn người, lúc đó số lượng request đến server của bạn sẽ rất nhiều, thời gian response chắc chắn sẽ lâu, vậy làm cách nào bạn có thể tối ưu được performance của server. Chắc chắn bạn sẽ nghĩ đến sẽ là thuê một server khỏe hơn.. =))

Nhưng không, đó không phải là cách duy nhất, trong bài viết này mình sẽ giới thiệu về cách hay hơn mà tiết kiệm chi phí hơn đó là Caching -  Bộ nhớ đệm - cũng như Redis - một công cụ tuyệt vời để implement Caching.

### 1: Cache là gì ?
Cache (Bộ nhớ đệm) là một bộ nhớ tạm thời mà những dữ liệu ở đó có thể cần dùng đến trong tương lai. Giúp cho tốc độ xử lý dữ liệu được nhanh chóng và chính xác hơn. Bất cứ khi nào ứng dụng của bạn cần truy xuất dữ liệu thì nó sẽ tìm trong Cache trước, nếu như không tìm thấy thì lúc đó nó mới truy xuất vào database.  Cache giúp giảm thời gian truy xuất dữ liệu trong database ngắn hơn.
Đặc điểm nổi bật :

- Dung lượng thấp
- Tốc độ xử lý nhanh

### 2: Redis là gì ?
- Có một công cụ tuyệt vời để hỗ trợ cho cache này chính là redis
- Vậy Redis là gì ? redis là một nơi lưu trữ dữ liệu, được dùng như một database, bộ nhớ đệm hoặc là nơi trung chuyển thông tin. Nó hỗ trợ nhiều cấu trúc dữ liệu khác nhau  như: strings, hashes, lists, sets, bitmaps ...

**Ưu điểm của redis:**
- Tại sao redis lại nhanh tới vậy ? bởi vì nó được viết bằng ngôn ngữ C, mà chúng ta đã biết tốc độ của C chạy như thế nào r đó.
- Không chỉ vậy Redis là một database thuộc dạng NoSQL Database.
- Hiện tại, nó đang được sử dụng bởi những gã khổng lồ công nghệ như GitHub, Weibo, Pinterest, Snapchat, Craigslist, Digg, StackOverflow, Flickr.
- So với việc bạn sử dụng dữ liệu lưu trữ trên cloud database thì việc lưu ở redis cache sẽ tiết kiệm chi phí hơn.
- Redis được hỗ trợ bởi rất nhiều ngôn ngữ lập trình như JavaScript, Java, Go, C, C++, C#, Python, Ruby, PHP, Objective-C, ...
- Cuối cùng và có lẽ là điểm rất rõ ràng, redis open source và ổn định, vì vậy tại sao ta lại ko chọn nó cơ chứ.

**Redis khác với NoSQL database như thế nào :**
![](https://images.viblo.asia/1f404579-f64e-4eda-bef0-b71ce22ce2ba.jpeg)

Như hình bên trên, ta có thể thấy NoSQL Database hầu hết các loại database này đều không định nghĩa cấu trúc như table hay hỗ trợ các câu query như SQL SELECT. NoSQL ít ràng buộc với schema hơn khi mà dữ liệu của nó hầu hết được lưu trữ dưới dạng JSON.

Còn Redis là một database theo kiểu Key - Value. Có nghĩa là một value sẽ được ánh xạ tương ứng với một key. Những value này có thể khác nhau hoàn toàn và không tuân theo bất cứ một cấu trúc cụ thể nào cả. Tuy nhiên, cách duy nhất để có thể lấy value là thông qua key của nó.

### 3: Một số câu lệnh hay dùng ?
Để có thể cài đặt Redis, bạn có thể làm theo chỉ dẫn trong phần [download](https://redis.io/download) này.
Ở đây chúng ta sử dụng redis CLI để thao tác với redis. Theo mặc định, Redis-CLI chạy trên cổng 6379.

SET (Setting a Key)
```
127.0.0.1:6379> SET foo "Hello World"
OK // setting a key
```

GET (Getting a Key)
```
127.0.0.1:6379> GET foo
"Hello World" // getting a key
```

DEL (Deleting a Key)
```
127.0.0.1:6379> GET foo
"Hello World" // getting a key
127.0.0.1:6379> DEL foo
(integer) 1 // key just got deleted
127.0.0.1:6379> GET foo
(nil) // since key is deleted therefore, result is nil.
```

SETEX (Setting a key with an expiry)

```
127.0.0.1:6379> SETEX foo 40 "I said, Hello World!"
OK // key has been set with 40 seconds as expiration
```

TTL (Total Time left for a key that has a timeout)
```
127.0.0.1:6379> TTL foo
(integer) 36 // 36 seconds left to timeout
```

PERSIST (Removing the timeout from key)
```
127.0.0.1:6379> PERSIST foo
(integer) 1 // turning the key from volatile to persistent (key won't expire)
```

RENAME (Renaming the current existing key)
```
127.0.0.1:6379> RENAME foo bar
OK // renaming the key 'foo' as bar
```
FLUSHALL (Flushing everything so far saved)
```
127.0.0.1:6379> flushall
OK // just got flushed
```

Vậy đó chỉ là phần giới thiệu về những gì bạn có thể làm với Redis. Redis cực kỳ mạnh mẽ và thậm chí tô cũng đang sử dụng hàng ngày để hiệu suất dữ liệu được truyền nhanh hơn giữa các ứng dụng.


### 4: Dùng redis với ruby
Gem mà hay dùng để hỗ trợ việc dùng redis bằng ruby đó chính là gem:  [gem "redis"](https://github.com/redis/redis-rb)

Để có thể cài đặt gem bạn lại vào rails c và gõ bundle install thôi ạ.

Sau đó bạn phải chạy redis-server ở một terminal khác để start server trước khi bắt đầu

Gem này cũng cung cấp những method tương tự giống như redis-cli, bạn có thể vào terminal và thử như sau
```
require 'redis'
redis = Redis.new(host: "localhost")
redis.set("a", 1)
# "OK"
redis.get("a")
# "1"
```

- Set expire cho cache
```
redis.setex("bacon", 10, 100)
```
trong đó: tham số thứ hai là số giây mà cái key này sẽ expire còn tham số thứ ba là value của nó.

- Để sắp xếp các key trong redis:

đầu tiên ta sẽ add các key vào trong redis
```
redis.zadd("popular_fruit", 10, "apple")
# true
redis.zadd("popular_fruit", 20, "banana")
# true
redis.zadd("popular_fruit", 30, "orange")
# true
```
Sau đó ta sắp xếp chúng bằng lệnh sau
```
r.zrevrange("popular_fruit", 0, 0)
# ["orange"]
# Bắt đầu từ thứ hạng cao nhất và lấy ra thằng đầu tiên

r.zrevrange("popular_fruit", 0, -1)
# ["orange", "banana", "apple"]
```

- Dữ liệu lưu trong redis không có cột, không có bảng, mọi thứ là một namespace. Làm thế nào để có thể tổ chức dữ liệu trong redis?
Ta sẽ sử dụng key của nó, và trong key đó một quy ước chung là sử dụng dấu hai chấm (:) để phân tách tên chung & phần cụ thể của key đó.
```
redis.set("fruit:1", "apple")
# OK
redis.set("fruit:2", "banana")
# OK
```

Sử dụng dấu hai chấm về bản chất không có gì thay đổi, Redis vẫn sẽ nhận diện đây là hai key khác nhau.

**GUI client tool for Redis**

Mặc dù Redis cung cấp công cụ CLI để hoạt động với Redis, ngoài ra mình suggest bạn nên sử dụng GUI: [redis desktop management](https://redisdesktop.com/) để dễ sử dụng, giao diện trực quan và dễ dàng thao tác với dữ liệu trên redis
Chúng ta sẽ có thể kết nối với máy chủ Redis đang chạy với tư cách là client và xem những gì bên trong cơ sở dữ liệu Redis.
![](https://images.viblo.asia/7b950837-dda5-4c11-8578-471f6ba37a8d.png)

Ở đây chúng ta có thể thấy các key hiện có và value của chúng trong cơ sở dữ liệu Redis. Hơn nữa, bạn sẽ có thể thêm khóa mới hoặc lọc được theo key.

Tài liệu tham khảo: https://medium.com/@chathuranga94/introduction-to-redis-348d9ccbfd0d
https://codeburst.io/redis-what-and-why-d52b6829813
