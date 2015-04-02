# 5. Active Record basics. Rails models. Databases and Rails app

#### Генерацыя мадэлі

    $ rails generate model Testimonial user_name:string feedback:text --no-test-framework

#### Усталёўка гема pg і канфігурацыя прыкладання

Перш за ўсё трэба вынесці gem pg з групы production.

    # Gemfile
    gem 'pg'

Магчыма, спатрэбіцца усталяваць дадатковую бібліятэку для канфігурацыі postgresql:

    $ sudo apt-get install libpq-dev

Не забудзьце выдаліць з гемфайла sqlite3 - ён больш не патрэбны. Усталёўваем:

    $ bundle

Далей трэба абнавіць канфігурацыйны файл базы дадзеных нашага прыкладання.

    # config/database.yml
    default: &default
      adapter: postgresql
      encoding: unicode
      pool: 5

    development:
      <<: *default
      database: course-app_development

    test:
      <<: *default
      database: course-app_test

    production:
      <<: *default
      database: course-app_production
      host: localhost

##### Усталёўка PostgreSQL

    $ sudo apt-get install postgresql

Пасля усталёўкі заходзім у кансоль пад юзерам postgres:

    $ sudo su postgres

Ствараем карыстальніка:

    $ createuser -s micrum

#### Стварэнне і міграцыя базы дадзеных

    $ rake db:create

Міграцыі:

    $ rake db:migrate

#### Rails console


    $ rails console

Create testimonial:

    > testimonial = Testimonial.new(user_name: 'Max Planck', feedback: 'the energy of oscillators in a black body is quantized')
    > testimonial.save

Find testimonial using Active Record

    > testimonial = Testimonial.find(1)
    > testimonial.user_name


#### Generate Student model

    rails generate model Student github_user:string --no-test-framework

#### New and show testimonial actions:

    # app/models/testimonial.rb
    class TestimonialsController < ApplicationController

      def index
      end

      def new
      end

      def show
        @testimonial = Testimonial.find(params[:id])
      end
    end


New testimonial view:

    #app/views/testimonials/new.html.erb
    <div class="page-header">
        <h1>New testimonial</h1>
    </div>


New and Show routes:

    resources :testimonials, only: [:index, :new, :show]

New and Show routes:

    #config/routes.rb
    resources :testimonials, only: [:index, :new, :show]

SHOW view:

    #app/views/testimonials/show.html.erb
    <div class="page-header">
        <p><%= @testimonial.user_name %></p>
        <p><%= @testimonial.feedback %></p>
    </div>

New testimonial form:

    #app/views/testimonials/new.html.erb
    <%= form_for(@testimonial) do |f| %>

      <%= f.label :user_name %>
      <%= f.text_field :user_name %>

      <%= f.label :feedback %>
      <%= f.text_field :feedback %>

      <%= f.submit "Create testimonial", class: "btn btn-large btn-primary" %>
    <% end %>

Update controller:

      def new
        @testimonial = Testimonial.new
      end

    def create
      @testimonial = Testimonial.new(testimonial_params)
      if @testimonial.save
        redirect_to @testimonial
      else
        render 'new'
      end
    end

      private

      def user_params
        params.require(:user).permit(:name, :email, :password,
                                   :password_confirmation)
      end


#### Validations

Student model validations:

    class Student < ActiveRecord::Base
      validates :github_user, presence: true, uniqueness: true
    end

Testimonial model validations:

    class Testimonial < ActiveRecord::Base
      validates :user_name, :feedback, presence: true
    end


[<< папярэдні занятак](4_lecture.md)
[наступны занятак >>](6_lecture.md)