+++
title = "Truy vấn N + 1 hoặc vấn đề bộ nhớ"
description = "N + 1 là gì và vấn đề là làm thế nào để giải quyết nó ?"
author = "DungHX"
date = "2020-08-25"
tags = ["N + 1", "ruby"]
categories = ["N + 1", "ruby"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/n+1.png"
  alt = "Truy vấn N + 1 hoặc vấn đề bộ nhớ"
  stretch = "horizontal"
+++

Đã bao giờ bạn tự hỏi tại sao ứng dụng của mình tải trang chậm hơn khi bạn cần tải những đoạn dữ liệu lặp đi lặp lại hay lấy dữ liệu từ một tập các dữ liệu? Câu trả lời có thể là vì bạn có vấn đề với N + 1 queries, điều đó làm chậm trang web của bạn đáng kể. Nhưng N + 1 là gì và vấn đề là làm thế nào để giải quyết nó?

### Bài toán:
![](https://images.viblo.asia/8b3ec6bb-a9c6-40b0-b59d-bb725f484fe3.jpg)


Ví dụ ta đang xây dựng một blog, trong blog có nhiều posts và mỗi post thì có nhiều comments. Đầu tiên ở mỗi trang index của chúng ta là hiển thị danh sách các post, mỗi post cần phải hiên thị title, một đoạn description ngắn, và thông tin số lượng comment của post đấy. Bạn có một model Post và model Tag model Comment. Để hiển thị ra các bài post :

Controller
```
@posts = Post.all.per_page(20).page(params[:page])
```
View:

```
<% @posts.each do |post| %>
  <h1><%= link_to(post, post.title) %></h1>
  <p><%= post.description %></p>
  <%= "#{post.comments.count} comments" %>
<% end %>

<%= pagination(@posts) %>
```
Vấn đề ở trên là gì? Chúng ta phải thực hiện một truy vấn duy nhất để trả về tất cả các bài đăng - @posts. Tiếp theo ở mỗi post sẽ gọi thêm 1 query đẻ tính số lượng comment ở mỗi post đấy. Như vậy ứng dụng của bạn đã bị N+1 query.

### Sử dụng Includes
Nếu bạn đã code Rails qua nhiều dự án thì có lẽ bạn đã gặp phải tình huống trên rất nhiều. Nếu bạn tìm kiếm Google, câu trả lời rất đơn giản - "sử dụng Includes". Một giải phát đơn giản là sử dụng includes như chúng ta hay làm để Active Record lấy ra tất cả các comments của các post có liên quan
```
# before
@posts = current_user.posts.per_page(20).page(params[:page])
```

```
#and after
@posts = current_user.posts.per_page(20).page(params[:page])
@posts = @posts.includes(:comments)
```
Sau khi thêm includes thì chúng ta chỉ có 2 query là:

```
select * from posts where user_id=? limit ? offset ?
```

```
select * from comments where post_id in ?
```

Vấn đề N+1 query đã được giải quyết, nhưng bây giờ vấn đề mà bạn gặp phải đó là  vấn đề bị tràn bộ nhớ.
Nếu ta có 20 bài đăng trên blog và mỗi bài đăng có 100 bình luận, thì truy vấn này sẽ trả về 2.000 active record object từ cơ sở dữ liệu của bạn. Active Record không biết bạn cần dữ liệu gì từ mỗi bình luận bài đăng, mà đây là vấn đề lớn vì chúng ta không cần nhiều đến thế, chúng ta chỉ cần lấy số lượng comment mà thôi.
Hình dung mỗi comment đều có nội dung khá dài, việc chúng ta load ra hết như thế gây thừa thãi tốn memory rất nhiều. điều tốt duy nhất có ở đây là nó đã giải quyết đươc vấn đề N+1 query tuy nhiên chúng ta bị vấn đề là sẽ bị tràn bộ nhớ.

N + 1 is bad, phân bổ bộ nhớ không cần thiết is worse
Vậy chúng ta làm sao để giải quyết được 2 vấn đề trên cùng lúc.
### Dùng Count cache
Counter cache là kỹ thuật để tăng performance cho application thông qua việc tiết kiệm số lần gọi đến SQL. Cách thực thi rất đơn giản nhưng đem lại hiệu quả khá cao. Follow là chúng ta sẽ thêm 1 cột comments_count trong bảng posts. Mỗi lần có sự thay đổi về số lượng comment ta sẽ update trường này. Việc gọi ra thì khá đơn giản

```
class Comment < ApplicationRecord
   belongs_to :post , counter_cache: count_of_comments
  #…
end
```

```
 <%= "#{post.count_of_comments} comments"  %>
```

Bây giờ N + 1 và phân bổ bộ nhớ không cần thiết đã được xử lý, nhưng....
Giả sử rằng bạn ko thể sử dụng Count cache cho một điều kiện duy nhất, Ví dụ yêu cầu hiện thị số lượng comment đã được approved và đang pending thì thế nào.
Trước đây chúng ta làm như thế này:

```
<%= "#{ post.comments.approved.count } approved comments"  %>
 <%= "#{ post.comments.pending.count } pending comments"  %>
```
Trong trường hợp này, model Comment có trường status và gọi comment.pending tương đương với việc thêm where(status: "pending"). chúng ta phải thêm 2 trương ở bảng post là post.count_of_pending_comments và post.count_of_approved_comments. Ok có vẻ thêm mới 2 trường không vấn đề gì nhưng với trường hợp phức tạp hơn nữa làm như vậy thiết kế database của chúng ta sẽ ko được tốt.
### Build Count Data in Hashed

Một cách khác là chúng ta sẽ tạo 1 hash với key là post_id và value là số lượng comments tương ứng. Việc tạo ra khá đơn giản chỉ với 1 query.

```
@posts = current_user.posts.per_page(20).page(params[:page])
@posts = @posts.includes(:comments)
```

Chúng tôi có thể loại bỏ includes và thay vào đó xây dựng two hashes. Active Record trả về giá trị hash khi chúng ta sử dụng group (). Trong trường hợp này, chúng tôi biết rằng chúng tôi muốn tạo quan hệ số lượng comment với mỗi post, vì vậy chúng tôi phải group by :post_id.

```
@posts = current_user.posts.per_page(20).page(params[:page])
post_ids = @posts.map(&:id)
@pending_count_hash   = Comment.pending.where(post_id: post_ids).group(:post_id).count
@approved_count_hash = Comment.approved.where(post_id: post_ids).group(:post_id).count
```

Để lấy ra số lượng comment chúng ta chỉ cần gọi

```
  <%= "#{ @approved_count_hash[post.id] || 0  } approved comments"  %>
  <%= "#{ @pending_count_hash[post.id] || 0 } pending comments"  %>
```

Tổng thể chúng ta chỉ cần 3 query, một để lấy ra các post và 2 query để lấy ra 1 hash chứa thông tin số lượng comment vs mỗi post tương ứng.

Sau khi tìm hiểu thì mình thấy đây là cách tốt nhất để giải quyết được vấn đề bộ nhớ và tránh được N+1 query.

Tổng kết thì tùy từng bài toán mà bạn có thể dùng những cách phù hợp để giải quyết, với bài toàn đơn giản chúng ta có thể dùng counter cache còn không có thể dùng cách trên or một cách nào khác nhưng phải đảm bảo được 2 vấn đề để ở trên để đảm bảo được perfomance tốt nhất cho ứng dung của bạn

Tài liêu tham khảo: https://blog.heroku.com/solving-n-plus-one-queries
