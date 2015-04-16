# 7. Аутэнтыфікацыя. Сессіі. Фільтры Action Controller. Flash-паведамленні

Ствараем мадэль ментара (выкладчыка курса):

    $ rails g model Mentor name:string password:digest --no-test-framework

Не забываем пра міграцыі:

    $ rake db:migrate

Валідацыі:
 
```ruby   
# app/models/mentor.rb
class Mentor < ActiveRecord::Base
  has_secure_password

  validates :name, presence: true, uniqueness: true
  validates :password, presence: true, length: { minimum: 6 }
end
```

Дадаем гем [bcrypt](https://github.com/codahale/bcrypt-ruby):

    #Gemfile
    gem 'bcrypt', '~> 3.1.7'

Пасля усталёўкі трэба будзе перазапусціць сервер.

Ствараем карыстальніка:
   
    $ Mentor.create(name: 'mentor_name', password: 'password')

Ствараем кантроллер для адміністрацыйнай старонкі ментара:

```ruby
# app/controllers/mentor_controller.rb
class MentorController < ApplicationController
  def index
  end
end
```

Дадаем адміністрацыйную старонку:

```ruby
<div class="page-header">
  <h1>Welcome, Admin!</h1>
</div>
```

Ствараем кантроллер сесій:

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def new
  end

  def create
    mentor = Mentor.find_by(name: params[:name])
    if mentor and mentor.authenticate(params[:password])
      session[:mentor_id] = mentor.id
      redirect_to mentor_url
    else
      redirect_to login_url, alert: 'Invalid username or password'
    end
  end

  def destroy
  end
end
```

Форма для увахода:

```html
# app/views/sessions/new.html.erb
<% if flash[:alert] %>
    <div class="alert alert-danger"><%= flash[:alert] %></div>
<% end %>
<%= form_tag({:action => 'create'}, {class: 'form-horizontal center'}) do %>
    <fieldset>
      <legend>Log In</legend>
      <div class="form-group">
        <%= label_tag :name, 'Name:', class: 'col-md-4 control-label' %>
        <div class="col-md-4">
            <%= text_field_tag :name, params[:name], class: 'form-control' %>
        </div>
      </div>
      <div class="form-group">
        <%= label_tag :password, 'Password:', class: 'col-md-4 control-label' %>
        <div class="col-md-4">
            <%= password_field_tag :password, params[:password], class: 'form-control' %>
        </div>
      </div>
      <div>
        <%= submit_tag 'Login', class: 'btn btn-large btn-primary' %>
      </div>
    </fieldset>
<% end %>
```

Маршруты:

```ruby
# config/routes.rb
get 'mentor' => 'mentor#index'

controller :sessions do
  get  'login' => :new
  post 'login' => :create
  delete 'logout' => :destroy
end
```

[<< папярэдні занятак](6_lecture.md)
[наступны занятак >>](8_lecture.md)
