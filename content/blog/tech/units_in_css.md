+++
title = "15 đơn vị CSS có thể bạn không biết ! em, rem, ex, cap, ch, ic..."
description = "Bạn đã bao giờ tự hỏi rằng không có màn hình nào mà bạn sở hữu có cùng kích thước không? Nhưng tất cả đều hiển thị các trang web. Vì vậy, các trang web cần phải được tạo kiểu theo cách thích nghi theo tỷ lệ tương đối với màn hình."
author = "DungHX"
date = "2020-08-15"
tags = ["css", "html"]
categories = ["css", "html"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/units_in_css.png"
  alt = "15 đơn vị CSS có thể bạn không biết ! em, rem, ex, cap, ch, ic..."
  stretch = "horizontal"
+++


Nếu bạn đang đọc bài này, tôi chắc rằng bạn biết ít nhất 5 cách xác định các phần tử HTML bằng cách sử dụng CSS. Vì vậy, 15 tùy chọn cho kích thước này chắc hẳn có những đơn vị mà bạn chưa từng biết đến.

**Đơn vị tương đối là gì và tại sao chúng ta cần chúng?**

Bạn đã bao giờ tự hỏi rằng không có màn hình nào mà bạn sở hữu có cùng kích thước không? Nhưng tất cả đều hiển thị các trang web. Vì vậy, các trang web cần phải được tạo kiểu theo cách thích nghi theo tỷ lệ tương đối với màn hình (hoặc bất kỳ phần tử HTML nào khác).

![](https://images.viblo.asia/99527cf9-0f20-4260-af0d-67f7eae95942.gif)

### **1: em unit**

So với kích thước của phần tử hiện tại (2em có nghĩa là gấp đôi kích thước phông chữ hiện tại). Một thuộc tính chiều dài phụ thuộc vào kích thước phông chữ.

```
.post {
   font-size: 24px;
   border-left: 0.25em solid #4a90e2;
}

/* The border-left-width will compute to 6px. */
```

Làm việc với 'em' có thể trở nên phức tạp vì nó có giá trị tương đối so với phần tử cha của nó. Trong trường hợp các phần tử lồng nhau, nó sẽ lộn xộn khi giá trị gốc cũng có cùng đơn vị 'em'

### 2.  rem (root em)

Cũng giống như em nhưng liên quan đến kích thước của phần tử gốc của tài liệu.

```
p {
    margin-left: 1rem;
}

h4 {
    font-size: 2rem;
}

/* Unlike the em, which may be different for each element, the rem is constant throughout the document */

```

Đối với kỹ thuật này, đó là cơ bản để thiết lập kích thước cơ bản của bạn, theo mặc định của trình duyệt thường là 16px.

### 3,4. vh and vw

Không giống như em và rem phụ thuộc vào kích thước  vh & vw phụ thuộc vào kích thước của khung nhìn.

1vh = 1% or 1/100th of viewport height

1vw = 1% or 1/100th of viewport width

Bạn có thể đã thấy một số trang web với kiểu chữ có quy mô đẹp mắt khi bạn thay đổi kích thước cửa sổ trình duyệt của mình, vw & vh đã làm điều đó.

### 5,6. vmax & vmin

Viewport max and viewport min : 1vw hoặc 1vh, tùy theo mức nào lớn hơn hoặc nhỏ hơn.

1vmax = 1% hoặc 1 / 100th  lớn hơn giữa 1vw hoặc 1vh

1vmin = 1% hoặc 1 / 100th  nhỏ hơn giữa 1vw hoặc 1vh

### 7. Good'ol %

Khá phổ biến,  khá rõ ràng % có liên quan đến yếu tố gốc của nó.

```
.post {
  font-size: 24px;
}
.post-category {
  font-size: 85%;
}
.post-title {
  font-size: 135%;
}
/*
Child elements with % font sizes...

  85%
  0.85 * 24 = 20.4 px
  135%
  1.35 * 24 = 32.4 px
*/
```

### 8,9. Experimental vi & vb

Các API thử nghiệm không có nghĩa là được sử dụng trong production như ở đó. Không phải bất kỳ hỗ trợ trình duyệt nào.

1vb = 1% kích thước của containing block, theo hướng trục khối của block axis.

1vh = 1% kích thước của containing block, theo hướng inline axis của phần tử gốc.

Nó không phải là rất phổ biến hoặc được hỗ trợ,nó là khá mới, bạn có thể khai thác thêm về nó trong CSS Value.

### 10,11. lh & rlh

Chúng giống như em & rem nhưng thay vì cỡ chữ, chúng phụ thuộc vào chiều cao dòng.

lh = Bằng với giá trị tính toán của thuộc tính chiều cao dòng của phần tử mà nó được sử dụng.

rlh = Bằng với giá trị tính toán của thuộc tính chiều cao dòng trên phần tử gốc.

### 12. ex

Đơn vị ** ex ** hiếm khi được sử dụng. Mục đích của nó là để thể hiện các kích thước phải liên quan đến chiều cao x của một phông chữ. Chiều cao x, gần bằng chiều cao của các chữ thường như a, c, m hoặc o.

### 13. cap

Đó là thước đo của cái gọi là chiều cao mũ. Thông số xác định giới hạn chiều cao bằng xấp xỉ bằng độ cao ** của chữ cái Latinh vốn.

### 14. ch

Đại diện cho ** chiều rộng của ký tự 0, hoặc chính xác hơn là số đo trước của chữ "0" (không, ký tự Unicode U + 0030) trong phông chữ của phần tử

### 15. 'ic' ideograph count

ic là tương đương với ch được đề cập ở trên. Đó là kích thước của chữ viết tay CJK (Trung Quốc / Nhật Bản / Hàn Quốc) * 水 * ("nước", U + 6C34), do đó, nó có thể được hiểu là "số ideograph".

Nguồn tham khảo: https://dev.to/bytegasm/15-css-relative-units-how-many-do-you-know-em-rem-ex-cap-ch-ic-6m
