https://www.sitepoint.com/create-a-chat-app-with-rails-5-actioncable-and-devise/

# Initial commit

```
rails new cable-chat -T
```
# Install devise, generate devise Users and views

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
