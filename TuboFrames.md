# TurboFrames 是什么
Turbo Frames是网页的独立部分，可以在不刷新页面和编写一行JavaScript的情况下在页面上追加、替换、删除、前置追加页面！

我们使用turbo_frame_tag帮助方法创建turbo frame，例如：
```ruby
<%= turbo_frame_tag "first_turbo_frame" do %>
  <div class="header">
    <h1>Quotes</h1>
    <%= link_to "New quote", new_quote_path, class: "btn btn--primary" %>
  </div>
<% end %>
# 在html中会生成一个turbo-frame标签
# 被包裹在这个标签里的内容，点击链接和提交表单都会被拦截，这部分页面独立与整个页面
```

# Turbo Frames 追寻的规则
1. 当点击Turbo Frame中的链接时，Turbo希望在目标页上有一个相同id的frame。然后目标页中相同id的frame内容替换
原始页面内相同id的frame.
比如：我在首页A中有id为first-turbo-frame的turbo-frame标签，这里面有个新增链接是跳转到新增B页面，那么当点击新增链接时，发送请求到后端，后端返回，turbo会在响应页面中找到id为first-turbo-frame的turbo-frame标签，然后将其内容替换到首页A中id为
first-turbo-frame的turbo-frame标签中。
2. 单击Turbo Frame内的链接时，如果在目标页中没有找到相同id的turbo-frame标签，则该frame会消失，并且控制台中会记录错误
Error: The response (200) did not contain the expected <turbo-frame id="first_turbo_frame"> and will be ignored.
3. turbo frame 中的链接和表单可以通过data-turbo-frame属性，指定替换的frame。
如：
```ruby
# quotes/index.html.erb
<main class="container">
  <%= turbo_frame_tag "first_turbo_frame" do %>
    <div class="header">
      <h1>Quotes</h1>
      <%= link_to "New quote",
                  new_quote_path,
                  data: { turbo_frame: "second_frame" },
                  class: "btn btn--primary" %>
    </div>
  <% end %>

  <%= turbo_frame_tag "second_frame" do %>
    <%= render @quotes %>
  <% end %>
</main>
# 第二个turbo-frame 名字 second_frame

# quotes/new.html.erb
<main class="container">
  <%= link_to sanitize("&larr; Back to quotes"), quotes_path %>

  <div class="header">
    <h1>New quote</h1>
  </div>

  <%= turbo_frame_tag "second_frame" do %>
    <%= render "form", quote: @quote %>
  <% end %>
</main>
```


# data-turbo-frame="_top"
_top是替换整个页面，并且浏览器网址也会替换
