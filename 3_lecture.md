# 3. Статычныя web-старонкi. HTML & CSS ў Rails. Bootstrap

#### HTML link

    # app/views/pages/home.html.erb
    <p>
      This is a demo application for Ruby on Rails
      <a href="https://github.com/micrum/course" target="_blank">course</a>
    </p>

#### Named routes

    # config/routes.rb
    get 'testimonials' => 'pages/testimonials'

#### Links in Rails

    # app/views/layouts/application.html.erb
    <%= link_to 'Home', root_path %>
    <%= link_to 'Course', course_path %>
    <%= link_to 'Testimonials', testimonials_path %>

#### Partials

    # app/views/shared/_header.html.erb
    <%= link_to 'Home', root_path %>
    <%= link_to 'Course', course_path %>
    <%= link_to 'Testimonials', testimonials_path %>

Rendering:

    # app/views/layouts/application.html.erb
    <%= render 'shared/header' %>

#### Images

    # app/views/pages/course.html.erb
    <h1>Course</h1>
    <%= image_tag 'poster.png', height: 500, alt: 'Ruby on Rails course' %>

#### CSS

    # app/assets/stylesheets/pages.scss
    h1 {
      color: green;
    }

    .centered {
      text-align: center;
    }

Using styles:

    #app/views/layouts/application.html.erb
    <div class="centered">
      <%= yield %>
    </div>

#### Bootstrap

    # Gemfile
    gem 'bootstrap-sass', '~> 3.3.3'

Import Bootstrap styles:

    # app/assets/stylesheets/styles.scss
    @import 'bootstrap';

Using Bootstrap:

    # app/views/shared/_header.html.erb
    <nav class="navbar navbar-inverse">
      <ul class="nav navbar-nav">
        <li><%= link_to 'Home', root_path %></li>
        <li><%= link_to 'Course', course_path %></li>
        <li><%= link_to 'Testimonials', testimonials_path %></li>
      </ul>
    </nav>

Update layout:

    #app/views/layouts/application.html.erb
    <div class="container text-center">
      <%= yield %>
     </div>

Update main view:

    # app/views/pages/home.html.erb
    <div class="page-header">
      <h1><%= @hello %></h1>
    </div>

    <div class="jumbotron">
      <p>This is a demo application for Ruby on Rails course</p>
      <%= link_to 'Visit course page', 'https://github.com/micrum/course',
                  target: '_blank', class: 'btn btn-primary btn-lg' %>
    </div>



[<< папярэдні занятак](2_lecture.md)
[наступны занятак >>](4_lecture.md)
