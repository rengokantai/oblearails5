#### oblearails5
#####Chapter 2. Rails on the Web
######Creating Your Own View
```
rails generate controller hello index
```
in config/routes.rb
```
get 'hello' => 'hello#index'
```
then visit
```
http://localhost:3000/hello
```
######Adding Some Data
app/controllers/hello_controller.rb
```
class HelloController < ApplicationController
  def index
    @message="Hello!"
    @count=3
    @bonus="This message came from the controller."
  end
end
```
######Adding Logic to the View
index.html.erb
```
<% @count.times do %>
  <p><%= @bonus %></p>
<% end %>
```
#####Chapter 3. Adding Web Style
######Creating a Layout for a Controller
hello.html.erb
```
<!DOCTYPE html>
<html>
<head>
  <title><%= @message %></title>
  <%= stylesheet_link_tag application, media: all, data-turbolinks-track => true %>
  <%= javascript_include_tag application, data-turbolinks-track => true %>
  <%= csrf_meta_tags %>
</head>
<body>
  <p>(using hello layout)</p>
  <%= yield %>   

</body>
</html>
```

the yield part will render index.html.erb part, so the rendered hello.html is
```
<!DOCTYPE html>
<html>
    <head>
    ...
    </head>
    <body>
        <p>(using hello layout)</p>
        <h1>Hello!</h1>
        <p>This is a greeting from app/views/hello/index.html.erb</p>
        <p>This message came from the controller.</p>
        ...
    </body>
</html>
```
######Choosing a Layout from a Controller
by default, hello controller will render hello.html.erb, to change that,
```
class HelloController < ApplicationController
  layout "standard_layout"
  def index
    ...
  end
end
```
this will render app/views/layouts/standard_layout.html.erb instead of app/views/layouts/hello.html.erb
