+++
title = "Design Patterns để refactor trong ruby"
description = "Design patterns là giải pháp thừờng đựoc áp dụng cho các vấn đề thường xảy ra trong thiết kế phần mềm"
author = "DungHX"
date = "2020-09-02"
tags = ["rails", "ruby", "Design Patterns"]
categories = ["vue js", "Design Patterns", "ruby"]
comments = true
removeBlur = false
[[images]]
  src = "img/2020/tech/design_patterns_ruby.jpg"
  alt = "Design Patterns để refactor trong ruby"
  stretch = "horizontal"
+++

## I: overview
Design patterns là giải pháp thừờng đựoc áp dụng cho các vấn đề thường xảy ra trong thiết kế phần mềm. Design patterns là các bản thiết kế được tạo sẵn mà bạn có thể tùy chỉnh để giải quyết vấn đề thiết kế định kỳ trong code của dự án mình đang làm.
Design patterns không phải là một đoạn code cụ thể mà ta có thể copy và paste vào code dự án của mình, mà là một khái niệm chung để giải quyết một vấn đề cụ thể.
Design patterns thường bị nhầm lẫn với các thuật toán, bởi vì cả hai khái niệm đều mô tả các giải pháp điển hình cho một số vấn đề đã biết. Trong khi một thuật toán luôn xác định một tập hợp các hành động rõ ràng có thể giải quyết đựoc một số mục tiêu, còn Design patterns là một mô tả về một giải pháp. Những designs này đã được thảo luận, chứng minh, kiểm thử, tối ưu và áp dụng bởi rất nhiều các senior. Vì thể chúng cũng nên tìm hiểu một chút và áp dụng để giải quyết những bài toán của dự án mà minh đang gặp phải nha >_<


## II:  Design Patterns để Refactor MVC Components trong Rails
### 1: Service Objects (and Interactor Objects)

Service Objects là design pattern được sử dụng nhiều nhất trong các Rails Application. Ý tưởng của design pattern này rất đơn giản - Nếu một phần logic nào đó không thực phù hợp để viết trong model hay controller thì tốt nhất bạn nên đưa nó vào service.
Service Objects thường được sử dụng trong các hành động phức tạp như
- Việc tính toán lương của nhân viên)
- Sử dụng API của các dịch vụ bên ngoài (external service)
- Không thuộc về bất kỳ 1 model nào
- Sử dụng nhiều model để xử lý (ví dụ: việc import dữ liệu từ 1 file excel bao gồm thêm data vào nhiều bảng khác nhau)

**Ví dụ**

Trong ví dụ dưới đây, công việc được thực hiện bới Strip service bên ngoài (external Strip service). Strip service tạo ra một Strip customer dựa trên email và source (token), và ràng buộc dịch vụ thanh toán (service payment) với tài khoản này.

**Vấn đề**

- Logic xử lý với external service được nằm trong controller
- Controller tạo dữ liệu cho một external service
- Điều này gặp khó khăn trong việc bảo trì và phân bổ controller
```
class ChargesController  exception
    flash[:error] = exception.message
    redirect_to new_charge_path
  end
end
```
Để giải quyết vấn đề này, ta cần đóng gói toàn bộ công việc trong một external service:

```
class ChargesController < ApplicationController
 def create
   CheckoutService.new(params).call
   redirect_to charges_path
 rescue Stripe::CardError => exception
   flash[:error] = exception.message
   redirect_to new_charge_path
 end
end

class CheckoutService
 DEFAULT_CURRENCY = 'USD'.freeze

 def initialize(options = {})
   options.each_pair do |key, value|
     instance_variable_set("@#{key}", value)
   end
 end

 def call
   Stripe::Charge.create(charge_attributes)
 end

 private

 attr_reader :email, :source, :amount, :description

 def currency
   @currency || DEFAULT_CURRENCY
 end

 def amount
   @amount.to_i * 100
 end

 def customer
   @customer ||= Stripe::Customer.create(customer_attributes)
 end

 def customer_attributes
   {
     email: email,
     source: source
   }
 end

 def charge_attributes
   {
     customer: customer.id,
     amount: amount,
     description: description,
     currency: currency
   }
 end
end
```

CheckoutService chịu trách nhiệm cho việc tạo tài khoản của Customer và thanh toán. Chúng ta đã giải quyết được vấn đề xử lý nhiều logic trong controller, tuy nhiên vẫn còn vấn đề khác chưa giải quyết. Điều gì sẽ xảy ra nếu external service ném ra 1 ngoại lệ (exception) (ví như khi credit card không hợp lệ) và chúng ta chuyển hướng người dùng đến 1 trang khác?

```
class ChargesController < ApplicationController
 def create
   CheckoutService.new(params).call
   redirect_to charges_path
 rescue Stripe::CardError => exception
   flash[:error] = exception.error
   redirect_to new_charge_path
 end
end
```

Để xử lý vấn đề này, chúng ta sẽ gọi CheckoutService và ngăn chặn exception với Interactor Object. Interactors được dử dụng để đóng gói logic nghiệp vụ (business logic). Mỗi Iteractor thường mô tả 1 quy tắc nghiệp vụ (business rule).

Interactor tương tự như Service Objects, nhìn chung chúng đều trả về một vài giá trị hiển thị trạng thái thực thi và một số thông tin khác (ngoài việc thực hiện các hành động). Thông thường cũng có sử dụng Service Objects bên trong Interactor Objects. Dưới đây là ví dụ về cách sử dụng mẫu thiết kế này
```
class ChargesController < ApplicationController
 def create
   interactor = CheckoutInteractor.call(self)

   if interactor.success?
 redirect_to charges_path
   else
 flash[:error] = interactor.error
 redirect_to new_charge_path
   end
 end
end

class CheckoutInteractor
 def self.call(context)
   interactor = new(context)
   interactor.run
   interactor
 end

 attr_reader :error

 def initialize(context)
   @context = context
 end

 def success?
   @error.nil?
 end

 def run
   CheckoutService.new(context.params)
 rescue Stripe::CardError => exception
   fail!(exception.message)
 end

 private

 attr_reader :context

 def fail!(error)
   @error = error
 end
end
```

Di chuyển tất cả các exception có liên quan đến 1 nơi, chúng ta có thể skinny controller. Bây giờ controller chỉ chịu trách nhiệm chuyển hướng người dùng đến các trang cho việc thanh toán thành công hay không thành công

### 2: Decorator Pattern

Trong OOP, decorator cho chúng ta khả năng mở rộng hành vi của một object cụ thể bằng cách cung cấp cho nó một số các method mở rộng. Ngoài ra Decorator Pattern cho phép chúng ta thêm bất kỳ behavior phụ trợ nào vào từng đối tương mà không ảnh hưởng đến các đối tượng khác trong cùng 1 class. Design pattern này được sử dụng rộng rãi để phân chia các chứ năng giữa các class khác nhau là một sự thay thế tốt cho các lớp con để tuân thủ nguyên tắc Single Responsibility Principle. - SRP

**Ví dụ**

Một vài vấn đề với View như:
- Hiển thị ảnh mặc định khi không cung cấp ảnh
- Hiển thi text mặc định nếu các trường không có giá trị
- Định dạng ngày tạo

**Vấn đề**: View chứa nhiều logic xử lý:

```
#/app/controllers/cars_controller.rb
class CarsController < ApplicationController
 def show
   @car = Car.find(params[:id])
 end
end

#/app/views/cars/show.html.erb
<% content_for :header do %>
 <title>
   <% if @car.title_for_head %>
     <%="#{ @car.title_for_head } | #{t('beautiful_cars')}" %>
   <% else %>
     <%= t('beautiful_cars') %>
   <% end %>
 </title>
 <% if @car.description_for_head%>
   <meta name='description' content= "#{<%= @car.description_for_head %>}">
 <% end %>
<% end %>

<% if @car.image %>
 <%= image_tag @car.image.url %>
<% else %>
 <%= image_tag 'no-images.png'%>
<% end %>
<h1>
 <%= t('brand') %>
 <% if @car.brand %>
   <%= @car.brand %>
 <% else %>
    <%= t('undefined') %>
 <% end %>
</h1>

<p>
 <%= t('model') %>
 <% if @car.model %>
   <%= @car.model %>
 <% else %>
    <%= t('undefined') %>
 <% end %>
</p>

<p>
 <%= t('notes') %>
 <% if @car.notes %>
   <%= @car.notes %>
 <% else %>
    <%= t('undefined') %>
 <% end %>
</p>

<p>
 <%= t('owner') %>
 <% if @car.owner %>
   <%= @car.owner %>
 <% else %>
    <%= t('undefined') %>
 <% end %>
</p>

<p>
 <%= t('city') %>
 <% if @car.city %>
   <%= @car.city %>
 <% else %>
    <%= t('undefined') %>
 <% end %>
</p>
<p>
 <%= t('owner_phone') %>
 <% if @car.phone %>
   <%= @car.phone %>
 <% else %>
    <%= t('undefined') %>
 <% end %>
</p>

<p>
 <%= t('state') %>
 <% if @car.used %>
   <%= t('used') %>
 <% else %>
   <%= t('new') %>
 <% end %>
</p>

<p>
 <%= t('date') %>
 <%= @car.created_at.strftime("%B %e, %Y")%>
</p>
```

Chúng ta có thể giải quyết vấn đề này với Draper decorating gem, chuyển tất cả logic sang các method CarDecorator.
```
#/app/controllers/cars_controller.rb
class CarsController < ApplicationController
 def show
   @car = Car.find(params[:id]).decorate
 end
end

#/app/decorators/car_decorator.rb
class CarDecorator < Draper::Decorator
 delegate_all
 def meta_title
   result =
     if object.title_for_head
       "#{ object.title_for_head } | #{I18n.t('beautiful_cars')}"
     else
       t('beautiful_cars')
     end
   h.content_tag :title, result
 end

 def meta_description
   if object.description_for_head
     h.content_tag :meta, nil ,content: object.description_for_head
   end
 end

 def image
   result = object.image.url.present? ? object.image.url : 'no-images.png'
   h.image_tag result
 end

 def brand
   get_info object.brand
 end

 def model
   get_info object.model
 end

 def notes
   get_info object.notes
 end

 def owner
   get_info object.owner
 end

 def city
   get_info object.city
 end

 def owner_phone
   get_info object.phone
 end

 def state
   object.used ? I18n.t('used') : I18n.t('new')
 end

 def created_at
   object.created_at.strftime("%B %e, %Y")
 end

 private

 def get_info value
   value.present? ? value : t('undefined')
 end
end
```

Và bây giờ kết quả của View gọn gàng và ko có bất kỳ logic nào
```
#/app/views/cars/show.html.erb
<% content_for :header do %>
 <%= @car.meta_title %>
 <%= @car.meta_description%>
<% end %>
​
<%= @car.image %>
<h1> <%= t('brand') %> <%= @car.brand %> </h1>
<p> <%= t('model') %> <%= @car.model %>  </p>
<p> <%= t('notes') %> <%= @car.notes %>  </p>
<p> <%= t('owner') %> <%= @car.owner %>  </p>
<p> <%= t('city') %> <%= @car.city %>    </p>
<p> <%= t('owner_phone') %> <%= @car.phone %> </p>
<p> <%= t('state') %> <%= @car.state %>   </p>
<p> <%= t('date') %> <%= @car.created_at%> </p>
```

### 3: Policy Objects

design pattern Policy Objects tương tự như Service Objects, nhưng nó phụ trách về các hoạt động đọc trong khi Service Objects phụ trách cho các thao tác ghi. Policy Objects đóng gói các validate phức tạp và có thể dễ dàng được thay thế bởi các Policy Objects khác bằng các quy tắc khác nhau. Ví dụ: có thể kiểm tra xem người dùng khách có thể truy xuất một số tài nguyên nhất định bằng cách sử dụng Policy Object khách hay không. Nếu người dùng là quản trị viên, chúng tôi có thể dễ dàng thay đổi Policy Object khách này thành Policy Object quản trị có chứa quy tắc quản trị viên.

Ví dụ:

Trước khi người dùng tạo dự án, Controller kiểm tra xem người dùng hiện tại có quyển là người quản lý hay không, họ có được phép tạo dự án hay không, số lượng dự án người dùng hiện tại có ở mức tối đa hay không và kiểm tra xem có các blocks khi tạo dự án trong Redis lưu trữ khóa / giá trị

Vấn đề:

- Chỉ có Controller biết về các Policy tạo dự án
- Controller chứa nhiều logic
```

class ProjectsController < ApplicationController
   def create
     if can_create_project?
       @project = Project.create!(project_params)
       render json: @project, status: :created
     else
       head :unauthorized
     end
   end

  private

  def can_create_project?
     current_user.manager? &&
       current_user.projects.count < Project.max_count &&
       redis.get('projects_creation_blocked') != '1'
   end

  def project_params
     params.require(:project).permit(:name, :description)
   end

  def redis
      Redis.current
    end
  end

  def User < ActiveRecord::Base
     enum role: [:manager, :employee, :guest]
  end

```

Để làm cho Controller giảm bớt logic, chúng tôi chuyển logic policy sang Model. Do đó, kiểm tra hoàn toàn được chuyển ra khỏi controller. Nhưng bây giờ class user đã biết về Redis và logic của class Project và do đó lại bị fat model

```
def User < ActiveRecord::Base
  enum role: [:manager, :employee, :guest]

  def can_create_project?
    manager? &&
      projects.count < Project.max_count &&
        redis.get('projects_creation_blocked') != '1'
  end

  private

  def redis
    Redis.current
  end
end

class ProjectsController < ApplicationController
  def create
    if current_user.can_create_project?
       @project = Project.create!(project_params)
       render json: @project, status: :created
    else
       head :unauthorized
    end
  end

  private

  def project_params
     params.require(:project).permit(:name, :description)
  end
end
```
Trong trường hợp này, chúng ta có thể làm cho Model và controller ko bị chứa quá nhiều logic bằng cách di chuyển logic này sang object policy

```
class CreateProjectPolicy
  def initialize(user, redis_client)
    @user = user
    @redis_client = redis_client
  end

  def allowed?
    @user.manager? && below_project_limit && !project_creation_blocked
  end

 private

  def below_project_limit
    @user.projects.count < Project.max_count
  end

  def project_creation_blocked
    @redis_client.get('projects_creation_blocked') == '1'
  end
end

class ProjectsController < ApplicationController
  def create
    if policy.allowed?
       @project = Project.create!(project_params)
       render json: @project, status: :created
     else
       head :unauthorized
     end
  end

  private

  def project_params
     params.require(:project).permit(:name, :description)
  end

  def policy
     CreateProjectPolicy.new(current_user, redis)
  end

  def redis
     Redis.current
  end
end

def User < ActiveRecord::Base
   enum role: [:manager, :employee, :guest]
end
```

Kết quả là controller và Model sẽ ko còn chứa nhiều code logic nữa. Policy object đóng gói logic kiểm tra quyền và tất cả các phụ thuộc bên ngoài được đưa từ Controller vào Policy object. Tất cả các class đều làm công việc riêng theo đúng chức năng của nó.

Tài liệu tham khảo

1, https://www.sitepoint.com/7-design-patterns-to-refactor-mvc-components-in-rails/
2, https://refactoring.guru/design-patterns/ruby
