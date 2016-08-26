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
- by default, hello controller will render hello.html.erb, to change that,
```
class HelloController < ApplicationController
  layout "standard_layout"
  def index
    ...
  end
end
```
this will render app/views/layouts/standard_layout.html.erb instead of app/views/layouts/hello.html.erb  

- using method reference. determined by method
```
class HelloController < ApplicationController

  layout :admin_or_user

  def index
    ...
  end

  private

  def admin_or_user
    if admin_authenticated
      "admin_screen"
    else
      "user_screen"
    end
  end
end
```
- only render with HTML format
```
layout "standard_layout", except: [:rss, :xml, :text_only]
layout "standard_layout", only: html
```
######Sharing Template Data with the Layout
create and yield named template.in index.html.erb,add
```

<% content_for(:list) do %>
  <ol>
  <% for i in 1..@count %>
    <li><%= @bonus %></li>
  <% end %>
  </ol>
<% end %>
```
in hello.index.erb
```
<%= yield :list %>
```
######Setting a Default Page
```
root to: "hello#index"
```
#####Chapter 4. Managing Data Flow: Controllers and Models
######Getting Started, Greeting Guests
```
rails new guestbook
cd guestbook
rails generate controller entries sign_in
```
app/controllers/entries_controller.rb
```
class EntriesController < ApplicationController
  def sign_in
    @name = params[:visitor_name]
  end
end
```
app/views/entries/sign_in.html.erb. The param will send to ocontroller
```
<%= form_tag action: 'sign_in' do %>
   <%= text_field_tag 'visitor_name', @name %></p>    //@name = default value
   <%= submit_tag 'Sign in' %>
<% end %>
```
config/routes.rb
```
get 'entries/sign_in' => 'entries#sign_in'
post 'entries/sign_in' => 'entries#sign_in'
```
see all routes
```
rails routes 
```
######Connecting to a Database Through a Model
```
rails generate model entry
```
generated files, add a column (note the syntax a.type :field), edit at db/migrate/[timestamp]_create_entries.rb
```
class CreateEntries < ActiveRecord::Migration
  def change
    create_table :entries do |t|
      t.string :name  #add column
      t.timestamps
    end
  end
end
```
then
```
rails db:migrate
```
get file at app/models/entry.rb
######Connecting the Controller to the Model, Storing data using the model
save data to  model
```
class EntriesController < ApplicationController
  def sign_in
    @name = params[:visitor_name]
    unless @name.blank?
      @entry = Entry.create({:name => @name})
    end
    @entries = Entry.all
  end
end
```
actually the second line is same as
```
@entry = Entry.new
@entry.name = @name
@entry.save
```
######Retrieving data from the model (see previous, @entries=Entry.all)
rails debug
```
<%= debug(params) %>
```
######Finding Data with ActiveRecord
```
rails c
Entry.find 1
Entry.first
Entry.where(name: "ke")
Entry.order(:name)
Entry.limit 3
Entry.limit(3).offset(2)
```
