+++
title = "Rubocop và run rubocop với GitHub Action"
description = "Làm thế nào giúp thời gian self review pull của bạn tiết kiệm thời gian hơn mà lại dễ dàng mà ko mất công rà lại từng dòng trước khi gủi pull ??"
author = "DungHX"
date = "2020-09-01"
tags = ["rails", "ruby", "rubocop", "git"]
categories = ["git", "ruby"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/rubocop.png"
  alt = "Rubocop và run rubocop với GitHub Action"
  stretch = "horizontal"
+++

Đã bao giờ mà bạn bị ăn cả một đống comment trước khi gửi pulls cho leader check, rồi bạn lại phải loay hoay fix nhưng vẫn không thể tìm ra hết lỗi trong pull rồi để rồi bạn lại bị chậm deadline ???

Điều này chắc hẳn đã xảy ra với bạn khi bạn làm mấy cái project để trainning, hay lúc mà bạn bị chậm deadline force push pull mà ko chịu review, bạn bị comment cả đống lỗi ngớ ngẩn như convention, ident, dấu câu, xuống dòng... Vậy làm thế nào mà bạn tránh được những cái lỗi ngớ ngẩn như vậy, giúp thời gian self review pull của bạn tiết kiệm thời gian hơn mà lại dễ dàng mà ko mất công rà lại từng dòng trước khi gủi pull.

Đó chính là rubocop, gem hỗ trợ mình check các convention trong code rails, Rubocop sử dụng các quy tắc được định sẵn để so sánh chúng với code của bạn rồi đưa ra các thông báo lỗi và từ đó mình có thể đảm bảo code không mắc phải những lỗi convention cơ bản.
### 1: Cài đặt

Cũng giồng như việc cài đặt gem khác trong ruby ta cũng có 2 cách

* Khai báo trong gem file rồi bundle install thôi:
```
gem 'rubocop', require: false
```
* Có thể chạy trực tiếp trên terminal:
```
gem install rubocop
```
### 2: Cách sử dụng
Vì Rubocop sử dụng các quy tắc được định sẵn để so sánh chúng với code của bạn nên có thể dễ dàng cho dev config theo convention riêng của dự án được:

Mọi cop của rubocop có thể được điều khiển trong file .rubocop.yml. Bạn có thể enable/disable các cops định nghĩa trong file.

Cấu trúc của file .rubocop.yml thường có dạng:
```

inherit_from: ../.rubocop.yml

 # Common configuration.
AllCops:
  Include:
    - "**/*.rb"
   ...
  Exclude:
    - "vendor/**/*"
    ...
```

Trong đó:

*  Include: chỉ check cops trong các file khai báo:
*  Exclude`: check cops ngoại trừ các file khai báo:

Tùy từng dự án, từng công ty mà các quy tắc khi code cũng sẽ khác nhau, chính vì thế rubocop không cố định hoàn toàn các quy tắc lỗi. Bạn hoàn toàn có thể chỉnh sửa config để rubocop bắt lỗi theo đúng những gì mà bạn muốn.
Cú pháp của 1 cop như sau:
```
rule_name:
  Description: 'a description of a rule'
  Enabled : (true or false)
  Key: Value
```
Trong đó:

* rule_name: là tên của cops
* Description : mô tả cop
* Enable : cho phép cop có được thực hiện hay không, thông thường theo mặc định sẽ là true.
* Key và Value : dùng để thêm thông tin cho một cop nào đó, ví dụ như các giá trị Max cho độ dài của dòng, của phương thức, của class

Sau khi mà bạn đã config các cops theo ý của bạn xong xuôi, thì làm thế nào để chạy rubocop, sẽ có rất nhiều cách khác nhau:

* Để chạy tất cả thư mục:
`rubocop`
* Chạy riêng một thư mục:
`rubocop app/`
* Chạy riêng từng file một:
`rubocop app/models/user.rb`
* Thậm chí bạn có thể chạy đồng thời các thư mục và các file khác nhau:
`rubocop app/models/user.rb app/decorators/`

Và để dễ dàng trong việc tra cứu tìm hiểu và biết các cops thuộc loại nào, Style hay Metrics, Rails... và tìm kiếm các rule cho convention của bạn, hay bạn có thể tra cứu convention đó rồi fix conventions trong lúc check rubocop mà bạn chưa biết, thì hãy tham khảo các config của rubocop tại link sau:
https://rubocop.readthedocs.io/en/latest/cops/

### 3:  Chạy Rubocop với GitHub Actions

GitHub Actions là công cụ mới mà github với hỗ trợ, có rất nhiều thứ hay ho mà có thể nghịch với chúng. Nhưng bây giờ mình sẽ chỉ hướng dẫn mn làm thế nào có thể chạy đc rubocop với cái GitHub Actions hay ho mà github mới hỗ trợ
Đầu tiên ta sẽ tạo một application rails mới để demo với bản  Ruby 2.6.5 và Rails 6.0.1.
Để chạy github action thì tất cả những gì chúng ta cần làm là thêm rubocop.yml vào file workflow:
```
mkdir -p .github/workflows
touch .github/workflows/rubocop.yml
```
Bạn nên đặt tên file là rubocop.yml bên trong .github/workflows.
```
# .github/workflows/rubocop.yml

name: Rubocop

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Rubocop Linter Action
      uses: andrewmcodes/rubocop-linter-action@v3.0.0.rc1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
Trong file chúng ta đang khai báo name là "Rubocop" và github action sẽ chạy nó trên phiên bản ub Ubuntu mới nhất bất cứ khi nào mà bạn push lên repo.

Tiếp theo, chúng ta sẽ kiểm tra kho lưu trữ bằng cách sử dụng action/checkout@v2, đây là bước kiểm tra repo của bạn $ GITHUB_WORKSPACE, để workflow có thể truy cập vào repo.

Sau đó, chúng ta sẽ chạy Rubocop Linter Action. Đối với hướng dẫn này và sẽ sử dụng phiên bản 3.0.0.rc1 cho action này.

Bước cuối cùng là lấy GITHUB_TOKEN mà github đã generated để set vào biến  env trong workflow

Bây giờ chúng ra sẽ tạo một cái repo trên github là devto-rubocop-linter-action-demo, sau khi repo đc tạo ta sẽ push code của mình lên nhánh master

```
git add .
git commit -m "first commit"
git remote add origin https://github.com/andrewmcodes/devto-rubocop-linter-action-demo.git
git push -u origin master
```

sau đó bạn mở tab github action trên repo của mình thì bạn sẽ thấy rõ:
![](https://images.viblo.asia/d662e68e-996b-48b6-828d-d0cbe444e8c7.png)

Sau đó bạn click vào "Build", github sẽ hiện thị cái log của action đó.
![](https://images.viblo.asia/935dc97d-e7e5-45ec-981e-ef5f65650a12.png)

Và bây giờ chúng ta có thể thấy tất cả các lỗi convention của action trên ứng dụng Rails, chỉ sử dụng mặc định Rubocop Linter Action và Rubocop mặc định. Hay thậm chí không cần thêm Rubocop vào Gemfile của mình!


**Advanced Configuration**
Ngoài config cơ bản, github action cũng support cho bạn những config khác mạnh hơn để hỗ trợ cho việc check pull request của bạn. Đó là 2 extension rubocop performance và rubocop rails


Đầu tiên bạn lại tạo 1 file config như trên:

```
# .github/config/rubocop_linter_action.yml

rubocop_extensions:
  - 'rubocop-rails'
  - 'rubocop-performance': '1.5.1'
```
Sau đó bạn commit và push lên master branch:

```
git add .
git commit -m "add rubocop and rubocop-linter-action config files"
git push --set-upstream origin advanced-rubocop-action-config
```

Sau đó trên github sẽ tạo pull request và lúc này bên tab github action bạn sẽ thấy sự khac biệt :
![](https://images.viblo.asia/4bf9e1a9-adc9-4f7c-a837-3c8c65f5167d.png)


Nếu bạn click vào 1 trong những cái lỗi mà rubocop bắt đc, github action sẽ show cho bạn ngay dòng code mà bạn bị rubocop bắt. Thật dễ dàng phải không, lúc này bạn chả cần phải cài bất gem nào trong gemfile mà vẫn có thể check convention cho pull của mình cả.

Nguồn tham khảo:
https://dev.to/andrewmcodes/run-rubocop-with-github-actions-4adp

https://github.com/rubocop-hq/rubocop/blob/master/manual/configuration.md

https://github.com/rubocop-hq/rubocop
