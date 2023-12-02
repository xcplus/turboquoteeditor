# Turbo Drive：表单响应必须重定向到另一个位置
```ruby
# 在创建或者更新的时候数据没有操作成功，返回的时候
# render :edit 或者 render :update 会在浏览器控制台中报如下错误：

# Form responses must redirect to another location

# 这是turbo-rails的原因，返回的时候加上status: :unprocessable_entity即可.
render :edit, status: :unprocessable_entity
render :update, status: :unprocessable_entity
# 这时候就可正常返回错误:
# POST http://localhost:3000/quotes 422 (Unprocessable Entity)
```

# Turbo Drive 是什么
Turbo Drive 是 Turbo的第一部分，默认安装在Rails 7应用中
```ruby
# Gemfile
gem "turbo-rails"
```

```javascript
// app/javascript/application.js
import "@hotwired/turbo-rails"
import "./controllers"
```
默认情况下，Turbo Driver为了提高应用程序的速度，把所有的链接点击和表单提交都转成了Ajax请求。
也就是我们默认是在开发一个单页应用。

当Turbo Drive拦截链接点击或表单提交时，对AJAX请求的响应将仅用于替换HTML页面的＜body＞。
在大多数情况下，当前HTML页面的＜head＞不会更改，从而显著提高了性能：当我们第一次访问网站时，只会发出一次下载字体、CSS和JavaScript文件的请求。

# Turbo Drive 是如何工作的
监听所有的链接和form表单，转换成ajax请求

# 禁用 Turbo Drive
- 方式一：（只是禁用个别的请求）
只需要在链接或者表单上添加如下属性即可:
```ruby
# data-turbo="false"
  <%= link_to "New quote",
                new_quote_path,
                class: "btn btn--primary",
                data: { turbo: false } %>
# 或者

<%= simple_form_for quote,
                    html: {
                      class: "quote form",
                      data: { turbo: false }
                    } do |f| %>
      ....
<% end %>
```

- 方式二：（全局禁用）
```js
// app/javascript/application.js
import { Turbo } from "@hotwired/turbo-rails"
Turbo.session.drive = false
```

# data-turbo-track="reload"
有时候我们更改了样式，这时候就需要重新加载head，可以使用
data-turbo-track="reload"这个属性，可以对比css或者js是否有更改，如果有更改turbo drive就会重载整个页面。
添加案例:
```ruby
# app/views/layouts/application.html.erb

<%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
<%= javascript_include_tag "application", "data-turbo-track": "reload", defer: true %>
```

# 改变Turbo Drive进度条的样式
```css
.turbo-progress-bar {
  background: linear-gradient(to right, var(--color-primary), var(--color-primary-rotate));
}
```