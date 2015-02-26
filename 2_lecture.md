# 2. Асновы працы і архітэктуры Rails прыкладання. MVC

## MVC

Генеруем кантроллер статычных старонак:

    $ rails generate controller Pages home course

Змяняем карнявы маршрут:

    # config/routes.rb
    root  'pages#home'
    
Дынамічны кантэнт:

    # app/controllers/pages_controller.rb
    def home
        @hello = 'Hi there!'
    end
    
Абнаўляем View:

    # app/views/pages/home.html.erb
    <h1><%= @hello %></h1>
    
Дадаем іменаваны маршрут:

    # config/routes.rb
    get 'course' => 'pages#course'
    
Дадаем новы маршрут кантроллера Pages:

    # config/routes.rb
    get 'pages/testimonials'
    
Ствараем action:

    # app/controllers/pages_controller.rb
    def testimonials
    end

Дадаем View:

    # app/views/pages/testimonials.html.erb
    <h1>Testimonials</h1>
    
## Разгортванне на Heroku

Абнаўляем Gemfile:

    # Gemfile
    # Use postgresql as the database for Active Record
    gem 'pg'

    group :production do
      #Heroku integration
      gem 'rails_12factor'
    end

Разгортванне:

    $ heroku login
    $ heroku create course-application
    $ git push heroku master

  
  

[<< папярэдні занятак](1_lecture.md)
[наступны занятак >>](3_lecture.md)
