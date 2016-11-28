https://www.sitepoint.com/create-a-chat-app-with-rails-5-actioncable-and-devise/

# >> Initial commit

```
rails new cable-chat -T
```
# >> Install devise, generate devise Users and views

*Gemfile*
```ruby
gem 'devise'
gem 'bootstrap', '~> 4.0.0.alpha3'
```
```
bundle install
```
*stylesheets/application.css.scss*
```
@import "bootstrap";
```
```ruby
rails g devise:install

rails g devise User

rails g devise:views

rails db:migrate
```
*application_controller.rb*
```ruby
before_action :authenticate_user!
```
# >> Add chat rooms
```
rails g model ChatRoom title:string user:references

rails db:migrate
```
*models/chat_room.rb*
```ruby
belongs_to :user
```

*models/users.rb*
```ruby
has_many :chat_rooms, dependent: :destroy
```
*chat_rooms_controller.rb*
```ruby
class ChatRoomsController < ApplicationController
  def index
    @chat_rooms = ChatRoom.all
  end

  def new
    @chat_room = ChatRoom.new
  end

  def create
    @chat_room = current_user.chat_rooms.build(chat_room_params)
    if @chat_room.save
      flash[:success] = 'Chat room added!'
      redirect_to chat_rooms_path
    else
      render 'new'
    end
  end

  private

    def chat_room_params
      params.require(:chat_room).permit(:title)
    end
end
```

*views/chat_rooms/index.html.erb*
```ruby
<h1>Chat rooms</h1>

<p class="lead"><%= link_to 'New chat room', new_chat_room_path, class: 'btn btn-primary' %></p>

<ul>
  <%= render @chat_rooms %>
</ul>
```
*views/chat_rooms/_chat_room.html.erb*
```ruby
<li><%= link_to "Enter #{chat_room.title}", chat_room_path(chat_room) %></li>
```
*views/chat_rooms/new.html.erb*
```ruby
<h1>Add chat room</h1>

<%= form_for @chat_room do |f| %>
  <div class="form-group">
    <%= f.label :title %>
    <%= f.text_field :title, autofocus: true, class: 'form-control' %>
  </div>

  <%= f.submit "Add!", class: 'btn btn-primary' %>
<% end %>
```
