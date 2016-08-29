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
#####Chapter 5. Accelerating Development with Scaffolding and REST
######A First Look at Scaffolding
Note: uppercase Person. Note: in db migration the syntax is ```t.string :name``` here is ```name:string```
```
rails generate scaffold Person name:string
```
######REST and Controller Best Practices, Toward a Cleaner Approach
CRUD in rails: show, create, update, and destroy

######Examining a RESTful Controller     (Needs a lot of review)
config/routes.rb:
```
resources :people
```
app/controllers/people_controller.rb
```
class PeopleController < ApplicationController
  before_action :set_person, only: [:show, :edit, :update, :destroy]   # must call set_person before calling show, edit,update,destroy

  # GET /people
  # GET /people.json
  def index
    @people = Person.all
  end

  # GET /people/1
  # GET /people/1.json
  def show
  end

  # GET /people/new
  def new
    @person = Person.new
  end

  # GET /people/1/edit
  def edit
  end

  # POST /people
  # POST /people.json
  def create
    @person = Person.new(person_params)

    respond_to do |format|
      if @person.save
        format.html { redirect_to @person, notice:
          'Person was successfully updated.' }
      else
        format.html { render :new }
        format.json { render json: @person.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /people/1
  # PATCH/PUT /people/1.json
  def update
    respond_to do |format|
      if @person.update(person_params)
        format.html { redirect_to @person, notice:                                    #@person = newly created person page
          'Person was successfully created.' }
        format.json { render :show, status: :ok, location: @person }
      else
        format.html { render :edit }
        format.json { render json: @person.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /people/1
  # DELETE /people/1.json
  def destroy
    @person.destroy
    respond_to do |format|
      format.html { redirect_to people_url,                                   #people_url = people list page
      notice: 'Person was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_person
      @person = Person.find(params[:id])
    end

    # Never trust parameters from the scary internet,
    #only allow the whitelist through.
    def person_params
      params.require(:person).permit(:name)                                 #sanity
    end
end
```
#####Chapter 6. Presenting Models with Forms
######Generating HTML Forms with Scaffolding
form_tag vs form_for.  
form_tag
```
<%= form_tag action: 'sign_in' do %>
   <%= text_field_tag 'visitor_name', @name %></p>    //@name = default value
   <%= submit_tag 'Sign in' %>
<% end %>
```
form_for using f to iterate
```
 <%= form_for(@person) do |f| %>
  <% if @person.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@person.errors.count, "error") %>
      prohibited this person from being saved:</h2>
      <ul>
      <% @person.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :name %>
    <%= f.text_field :name %>
  </div>
  <div class="field">
    <%= f.label :birthday %>
    <%= f.date_select :birthday %>
  </div>
  <div class="field">
    <%= f.label :favorite_time %>
    <%= f.time_select :favorite_time %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```
If dont want to use f,
```
<%= text_area :person, :description %>
instead of:
<%= f.text_area :description %>
```
######Creating Radio Buttons
tamplate
```
 <fieldset>
    <legend>Country</legend>
      <%= f.radio_button :country, 'USA' %> <%= f.label "person_country_usa",
      "USA" %><br>
      <%= f.radio_button :country, 'Canada' %> <%= f.label "person_country_canada",
      "Canada" %><br>
      <%= f.radio_button :country, 'Mexico' %> <%= f.label "person_country_mexico",
      "Mexico" %><br>
  </fieldset>
```
will render
```
  <fieldset>
  <legend>Country</legend>
  <input id="person_country_usa" name="person[country]" type="radio"
value="USA" />
  <label for="person_country_usa">USA</label><br>
  <input id="person_country_canada" name="person[country]" type="radio"
  value="Canada" />
  <label for="person_country_canada">Canada</label><br>
  <input id="person_country_mexico" name="person[country]" type="radio"
  value="Mexico" />
  <label for="person_country_mexico">Mexico</label><br>
</fieldset>
```
or not assign by hand, sort list
```
  <% nations = {'Canada' => 'Canada', United Kingdom' => 'UK' }%>

<fieldset>
  <legend>Country</legend>
  <% list = nations.sort
  list.each do |x| %>
    <%= f.radio_button :country, x[1] %>
    <label for="<%= ("person_country_" + x[1].downcase) %>">
    <%= x[0] %></label><br>
  <% end %>
</fieldset>
```
######Creating Selection Lists
syntax: actualshow, value
```
<%= f.label :country %><br>
<%= f.select :country, [ ['Canada', 'Canada'],
                         ['United Kingdom', 'UK']]%>
```
will render
```
<p>
  <label for="person_country">Country</label><br>
  <select name="person[country]" id="person_country">
  <option value="Canada">Canada</option>
  <option value="UK">United Kingdom</option>
</p>
```
assign a selected value
```
<%= f.select :country, [ ['Canada', 'Canada'],
                         ['United Kingdom', 'UK']], :selected=>'UK'%>
```
set sort lists from hash
```
<% nations = {  'Canada' => 'Canada','United Kingdom' => 'UK' }%>
<p>
  <%= f.label :country %><br>
  <% list = nations.sort %>
  <%= f.select :country, list %>
</p>
```
######Creating Helper Methods
```
<% nations = {'Canada' => 'Canada', 'United Kingdom' => 'UK' }%>
<%= buttons(:person, :country, nations) %>
```
create buttons method in people_helper.rb
```
  module PeopleHelper

    def buttons(model_name, target_property,button_source)
         html=''
         list = button_source.sort
         html << '<fieldset><legend>Country</legend>'
         list.each do |x|
           html << radio_button(model_name, target_property, x[1])
           html << (x[0])
          html << '<br>'
        end
        html << '</fieldset>'
        return html.html_safe
   end
  end
```
#####Chapter 7. Strengthening Models with Validation
######The Power of Declarative Validation
A error template
```
<% if person.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(person.errors.count, "error") %> prohibited this person
      from being saved:</h2>

      <ul>
      <% person.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
```
######Customizing the Message
[validata_prasence_of for enum](http://stackoverflow.com/questions/33513315/validates-presence-of-fails-with-enum)
```
validates_presence_of :secret,
    message: "must provide secret"
    
# password length 6-12
  validates_length_of :secret, in: 6..12

# at least one number
  validates_format_of :secret, with: /[0-9]/,
    message: "must contain at least one number"
```
######Limiting Choices
```
validates_inclusion_of :country, in: ['Canada','UK'],
    message: "must be one of Canada, UK"
```
(tbc)




#####Chapter 13. Sessions and Cookies
######Getting Into and Out of Cookies
simple cookie
```
cookies[:name] = @name
```
set cookie to path
```
cookies[:name] = { value: @name, path: '/entry' }
```
all atrributes
```
:value
:domain
:path
:expires
:secure
:http_only
```
