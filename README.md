## 链接
[turbo rails tutorial](https://www.hotrails.dev/turbo-rails)

### Turbo Drive
默认情况下，Turbo Drive 通过将所有链接点击和表单提交转换为 AJAX 请求来加速我们的 Ruby on Rails 应用程序。

#### Turbo Drive 是怎么工作的？
Turbo Drive 的工作原理是拦截链接上的“点击”事件和表单上的“提交”事件。
每次单击链接时，Turbo Drive 都会拦截“单击”事件，通过将链接单击通常会触发的 HTML 请求转换为 AJAX 请求来覆盖默认行为。

当 Turbo Drive 收到响应时，它会将当前页面的 <body> 替换为响应的 <body>，在大多数情况下保持 <head> 不变

无效的表单提交必须返回 422 状态代码，以便 Turbo Drive 替换页面的 <body> 并显示表单错误。 Rails 中 422 状态代码的别名是：unprocessable_entity。

这就是为什么自 Ruby on Rails 7 起，脚手架生成器会在由于无效表单提交而无法保存资源时将 status::unprocessable_entity 添加到 #create 和 #update 操作。

#### 禁用 Turbo Drive
要在链接或表单上禁用 Turbo Drive，我们需要在其上添加 data-turbo="false" 数据属性。

也可以为整个应用程序禁用 Turbo Drive，如下：
```js
// app/javascript/application.js
import { Turbo } from "@hotwired/turbo-rails"
Turbo.session.drive = false
```
不建议全局关闭。

#### data-turbo-track="reload"可以重载页面
比如我们增加了新的css并且需要使用的人重载页面才可以加载


### Turbo Frame
Turbo Frames 是网页的独立部分，无需完整的页面刷新和编写一行 JavaScript 即可附加、前置、替换或删除！

Turbo Frames 的一些规则:
1. 当点击 Turbo Frame 中的链接时，如果目标页面上有相同 id 的 frame，Turbo 会将源页面 Turbo Frame 的内容替换为目标页面 Turbo Frame 的内容。
2. 当点击 Turbo Frame 内的链接时，如果目标页面上没有相同 id 的 Turbo Frame，Turbo 将从源页面中删除 Turbo Frame 的内容，并且在浏览器控制台中报错误 没有匹配的<turbo-frame id="name_of_the_frame">；
3. 由于 data-turbo-frame 数据属性，链接可以指向它不直接嵌套的 Turbo Frame。在这种情况下，与源页面上的 data-turbo-frame 数据属性具有相同 id 的 Turbo Frame 将被替换为与目标页面上的 data-turbo-frame 数据属性具有相同 id 的 Turbo Frame。

注意:
data-turbo-frame="_top", 使用“_top”关键字时，页面的 URL 会更改为目标页面的 URL, 替换整个页面，这是与使用常规 Turbo Frame 时的另一个不同之处。

turbo_stream 响应的方法，可以执行以下操作：
```ruby
  # 删除Turbo Frame
  turbo_stream.remove

  # 在列表的开头/结尾插入 Turbo Frame
  turbo_stream.append
  turbo_stream.prepend

  # Turbo Frame 之前/之后插入一个 Turbo Frame
  turbo_stream.before
  turbo_stream.after

  # 更新或替换 Turbo Frame 的内容
  turbo_stream.update
  turbo_stream.replace
```

通过结合使用 Turbo Frames 和新的 TURBO_STREAM 格式，我们将能够对网页的各个部分执行精确的操作，而无需编写一行 JavaScript，从而保留网页的状态。

### turbo_stream_from
例如想在表User数据创建之后，想把这个数据即时更新到订阅的页面上则可以通过下面的例子操作：
1. 首页在A页面上添加订阅,以及A页面上该追加的地方
```ruby
# 订阅代码
<%= turbo_stream_from "users" %>

# 对应追加的地方
<%= turbo_frame_tag "user_list" %>
<%= render @users %>
<% end %>
```
上面代码订阅了“users”流。
2. 在表中添加创建之后推送代码:
```ruby
class User < ApplicationRecord
  after_create_commit -> { broadcast_prepend_to "users", partial: "users/user", locals: { user: self}, target: "user_list }
end
```
这时候就会在创建成功后自动把数据插入到`user_list`内部的最前方.