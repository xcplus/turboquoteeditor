# TurboStream
```ruby
<%= button_to "Delete",
                  quote_path(quote),
                  method: :delete,
                  class: "btn btn--light" %>
# 这个会发送格式是 TURBO_STREAM的请求

# 针对控制器则处理一个 TURBO_STREAM的请求
respond_to do |format|
  format.turbo_stream
end
# 针对view层返回则是
# 文件名 destroy.turbo_stream.erb
# 内容
<%= turbo_stream.remove "quote_#{@quote.id}" %>
# 还有其他turbo_stream方法
# Remove a Turbo Frame
# turbo_stream.remove

# # Insert a Turbo Frame at the beginning/end of a list
# turbo_stream.append
# turbo_stream.prepend

# # Insert a Turbo Frame before/after another Turbo Frame
# turbo_stream.before
# turbo_stream.after

# # Replace or update the content of a Turbo Frame
# turbo_stream.update
# turbo_stream.replace
```

通过Turbo Frames和新的Turbo_STREAM格式的组合，我们将能够对我们的网页片段执行精确的操作，而无需编写一行JavaScript，从而保持我们网页的状态。


# Turbo Stream即时更新
在创建之后立即更新:
```ruby
class Quote < ApplicationRecord
  after_create_commit -> {
    # 在创建之后，broadcast_preprend_to方法会通过prepend动作以Turbo Stream格式
    # 渲染quotes/_quote.html.erb到页面中有id是quotes的turbo-frame中，后面的target可选
    broadcast_prepend_to "quotes",
                          partial: "quotes/quote",
                          locals: { quote: self },
                          target: "quotes"
                        }
    # 唯一不同的是这个html片段会以websocket发送，而不是ajax响应个
    # 上面的broadcast_prepend_to的简写和语法糖
    # after_create_commit -> { broadcast_prepend_to "quotes" }
end
```

为了让我们的用户订阅 “quotes” 流，我们需要在 Quotes#index 视图中指定它:
```ruby
<%= turbo_stream_from "quotes" %>
```

# 使用ActiveJob异步广播
```ruby
class Quote < ApplicationRecord
  # All the previous code

  after_create_commit -> { broadcast_prepend_later_to "quotes" }
  after_update_commit -> { broadcast_replace_later_to "quotes" }
  after_destroy_commit -> { broadcast_remove_to "quotes" }
end
```

# 更多的语法糖
```ruby
class Quote < ApplicationRecord
  # after_create_commit -> { broadcast_prepend_later_to "quotes" }
  # after_update_commit -> { broadcast_replace_later_to "quotes" }
  # after_destroy_commit -> { broadcast_remove_to "quotes" }
  # 上面三个方法可以用下面的替代
  broadcasts_to ->(quote) { "quotes" }, inserts_by: :prepend
end
```


# 了解 Turbo Streams 和安全性


