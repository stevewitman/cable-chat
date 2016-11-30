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
# >> Add messages
```
rails g model Message body:text user:references chat_room:references

rails db:migrate
```
*models/chat_room.rb*

```ruby
has_many :messages, dependent: :destroy
```
*models/users.rb*

```ruby
has_many :messages, dependent: :destroy
```
*models/message.rb*
```ruby
belongs_to :user
belongs_to :chat_room
```
*chat_room_controller.rb*
```ruby
def show
  @chat_room = ChatRoom.includes(:messages).find_by(id: params[:id])
end
```
*views/chat_rooms/show.html.erb*
```ruby
<h1><%= @chat_room.title %></h1>

<div id="messages">
  <%= render @chat_room.messages %>
</div>
```
*views/messages/_message.html.erb*
```ruby
<div class="card">
  <div class="card-block">
    <div class="row">
      <div class="col-md-1">
        <%= gravatar_for message.user %>
      </div>
      <div class="col-md-11">
        <p class="card-text">
          <span class="text-muted"><%= message.user.name %> at <%= message.timestamp %> says</span><br>
          <%= message.body %>
        </p>
      </div>
    </div>
  </div>
</div>
```
*models/user.rb*
```ruby
def name
  email.split('@')[0]
end
```
*models/message.rb*
```ruby
def timestamp
  created_at.strftime('%H:%M:%S %d %B %Y')
end
```
*application_helper.rb*
```ruby
module ApplicationHelper
  def gravatar_for(user, opts = {})
    opts[:alt] = user.name
    image_tag "https://www.gravatar.com/avatar/#{Digest::MD5.hexdigest(user.email)}?s=#{opts.delete(:size) { 40 }}",
              opts
  end
end
```
*application.css.scss*
```css
#messages {
  max-height: 450px;
  overflow-y: auto;
  .avatar {
    margin: 0.5rem;
  }
}
```
*config/routes.rb*
```ruby
resources :chat_rooms, only: [:new, :create, :show, :index]
root 'chat_rooms#index'
```
