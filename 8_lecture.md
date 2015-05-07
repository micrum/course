# 8. Аўтарызацыя. Фільтры Action Controller. CRUD

## Аўтарызацыя у Rails. Фільтры Action Controller

На [мінулым занятку](7_lecture.md) мы навучыліся ствараць сессіі. Але таксама
трэба даваць магчымасць карыстальніку выдаляць сессіі. Гэта прынцыповае пытанне
бяспекі. Уявіце, што б было, калі б у вэб-прыкладаннях не было магчымасці
скончыць сессію (выйсці)? Выдаліць сессію у Rails даволі проста:

```ruby
# app/controllers/sessions_controller.rb
def destroy
  session[:mentor_id] = nil
  redirect_to root_url
end
```

Мы проста перадаем значэнне `nil` запісу id ментара у сесіі карыстальніка.

У нас ужо ёсць усе патрэбныя маршруты для кіравання сесіямі. Давайце абнавім
навігацыйную панель, каб карыстальніку было зручна кіраваць сесіямі:

```ruby
# app/views/shared/_header.html.erb
<nav class="navbar navbar-inverse">
  <ul class="nav navbar-nav navbar-left">
    <li><%= link_to 'Home', root_path %></li>
    <li><%= link_to 'Course', course_path %></li>
    <li><%= link_to 'Testimonials', testimonials_path %></li>
  </ul>

  <% if session[:mentor_id] %>
      <ul class="nav navbar-nav pull-right">
        <li><%= link_to 'Admin', mentor_path %></li>
        <li><%= button_to 'Logout', logout_path, method: :delete, class: 'btn navbar-btn' %></li>
      </ul>
  <% else %>
      <%= link_to 'Login', login_path, class: 'btn btn-default navbar-btn pull-right' %>
  <% end %>
</nav> 
```

Тут мы правяраем ці прысутнічае id ментара у дадзеных сесіі. Калі так, значыць
карыстальнік ужо ўвайшоў на сайт і мы адлюстроўваем кнопку выхада, якая выклікае
дзеянне **destroy()** кантролера Sessions. Таксама адлюструем спасылку на
адміністрацыйную старонку. У адваротным выпадку паказаем кнопку увахода.

Мы ужо можам вызначыць, ці прайшоў карыстальік аўтэнтыфікацыю, а значыць самы
час заняцца аўтарызацыяй. Мы абмяжуем правы доступа да нашай пакуль што поўнасцю
даступнай адміністрацыйнай старонкі.

Мы зробім гэта з дапамогай фільтра. Фільтр - гэта метад, які можа перарываць
(перахватваць) цыкл запыта. Напрыклад, метад фільтра **before_action()**
запускаецца перад выкананнем дзеянняў кантроллера.

Зараз мы створым метад, які будзе правяраць, ці аўтарызаваны карыстальнік. Мы
змясцім такі метад-фільтар у Application кантролер, для таго, каб можна была
выкарыстаць яго у розных кантролерах (бо усе нашы кантролеры спадкуюцца ад
гэтага кантролера).

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception

  protected

  def authorize
    unless Mentor.find_by(id: session[:mentor_id])
      redirect_to login_url, notice: 'Please log in'
    end
  end
end
```

Тут усё проста: калі ў сесіі карыстальніка не захоўваецца id ментара, значыць ён
не аўтарызаваны і замест дзеяння, для якога прымяняеццца фільтр, адбываецца
перанакіраванне на старонку увахода.

Зараз мы прымянім гэты метад да кантролера Ментара. 

    # app/controllers/mentor_controller.rb
    before_action :authorize

Даданне гэтага радка перахоплівае усе запыты да усіх дзеянняў кантроллера. У
нашым выпадку толькі адно дзеянне - **index()** (яму адпавядае адміністрацыйная старонка).

Паспрабуйце зараз зайсці на старонку */mentor* - нічога не атрымаецца без
аўтарызацыі. Падрабязней пра фільтры -
[тут](http://guides.rubyonrails.org/action_controller_overview.html#filters).

Мы рэалізавалі простую аўтарызацыю: вызначаем ролю карыстальніка і паказваем ці
не паказваем старонку у залежнасці ад гэтай ролі, якая вызначаецца праз
аўтэнтыфікацыю. А што наконт узаемадзеяння з базай дадзеных?

Натуральна, мы можам абмежаваць доступ не толькі да дзеянняў, якія рэалізуюцца
праз GET-запыты, але і да астатніх. У нашым прыкладанні напісанне водгукаў не
патрабуе аўтэнтыфікацыі, а значыць высокая верагоднасць атрымання спам-паведамленняў.
Таму варта даць магчымасць адміну выдаляць такія запісы.

Перш за ўсё дададзім маршрут destroy:

    # app/config/routes.rb
    resources :testimonials, only: [:index, :new, :show, :create, :destroy]

І дзеянне:

    # app/controllers/testimonials_controller.rb
    def destroy
      Testimonial.find(params[:id]).destroy
      redirect_to testimonials_url
    end

Зараз можна дадаць кнопку выдалення на старонку водгука:

```ruby
# app/views/testimonials/show.html.erb
<div class="page-header">
  <h2><%= @testimonial.user_name %></h2>
  <q><%= @testimonial.feedback %></q>
</div>
<% if session[:mentor_id] %>
    <%= link_to 'delete', @testimonial, method: :delete, class: 'btn btn-danger',
                data: { confirm: "You sure?" } %>
<% end %>
```

Кнопка не будзе адлюстроўвацца для неаўтарызаваных карыстальнікаў, але для
поўнай бяспекі мы павесім яшчэ фільтр на дзеянне **destroy()**:

    #app/controllers/testimonials_controller.rb
    before_action :authorize, only: :destroy

## CRUD

Мы ужо ведаем, як рэалізаваць аўтарызацыю, таму, нарэшце можна прыступіць да
стварэння RESTful рэсурсаў. Зараз можна не баяцца даваць аўтарызаванаму
карыстальніку узамадзейнічаць з базай данных. Мы ужо стварылі мадэль Student.
Давайце дазволім нашаму ментару весці улік студэнтаў, выдаляць, абнаўляць,
ствараць новыя запісы. Пры гэтым дадзім доступ неаўтарызаваным карыстальнікам
толькі да дзеяння *index* са спісам студэнтаў і *show* з канкрэтным запісам.

Давайце дададзім паўнавартасны рэсурсны маршрут. Такі маршрут злучае HTTP-метады
з дзеяннямі кантролера. Пры гэтым дзеянні адлюстроўваюцца з аперацыямі у базе дадзеных.

    # config/routes.rb
    resources :students

Каб было наглядней, паглядзіце гэтую узаемасувязь самі:

    $ rake routes

Зараз створым кантролер і дададзім дзеянні, аналагічныя кантролеру водгукаў:

```ruby
# app/controllers/students_controller.rb
class StudentsController < ApplicationController

  before_action :authorize, except: [:index, :show]

  def index
    @students = Student.all
  end

  def new
    @student = Student.new
  end

  def show
    @student = Student.find(params[:id])
  end

  def create
    @student = Student.new(student_params)
    if @student.save
      redirect_to @student
    else
      render 'new'
    end
  end

  def edit
    @student = Student.find(params[:id])
  end

  def update
    @student = Student.find(params[:id])
    if @student.update_attributes(student_params)
      redirect_to @student
    else
      render 'edit'
    end
  end

  def destroy
    Student.find(params[:id]).destroy
    redirect_to students_url
  end

  private

  def student_params
    params.require(:student).permit(:github_user, :name)
  end
end
```

Нічога новага. Тут мы выкарыстоўваем фільтр на усіх дзеяннях, акрамя *index*
і *show*. Таксама паглядзіце на дзеянне *update*. Для таго каб абнавіць аб'ект,
мы спачатку выклікаем форму (дзеянне *edit*). У гэтым дзеянні мы кладзем у
зменную экземпляра аб'ект класа студэнт, які знаходзім у базе па id (id
перадаецца як параметр у URL). Затым адлюстроўваем форму і праз форму ужо можам
выканаць дзеянне **update()**, якое і абнавіць запіс у базе дадзеных.

Адлюстраванне дзеянняў на аперацыі базы дадзеных - гэта CRUD (create, read, update, destroy).

Каб рашыць нашую задачу, засталося толькі дадаць адпаведныя views. Усё
трывіяльна, па аналогіі з водгукамі:

Адлюстроўваем спіс студэнтаў (кнопку 'new student' паказваем толькі ментару):

```ruby
# app/views/index.html.erb
<div class="page-header">
  <h1>Students</h1>
</div>

<% if session[:mentor_id] %>
    <div class="panel panel-default">
      <div class="panel-body">
        <%= link_to 'Add new student', new_student_path, class: 'btn btn-primary btn-lg' %>
      </div>
    </div>
<% end %>

<div class="list-group">
  <%= render @students %>
</div>
```

Частковы шаблон:

```ruby
# app/views/students/_student.html.erb
<a href="<%= student_path(student) %>" class="list-group-item">
    <b><%= student.name %></b>
    <p>Github username: <%= student.github_user %></p>
</a>
```

Старонка студэнта са спасылкай на дзеянне edit:

```ruby
# app/views/students/show.html.erb
<div class="page-header">
  <h2><%= @student.name %></h2>
  <p><%= @student.github_user %></p>
</div>

<% if session[:mentor_id] %>
    <%= link_to "edit", edit_student_path(@student), class: 'btn btn-default' %>
    <%= link_to 'delete', @testimonial, method: :delete, class: 'btn btn-danger',
                data: { confirm: "You sure?" } %>
<% end %>
   

```

Спасылкі на рэдагаванне і выдаленне студэнтаў паказваем толькі аўтарызаванаму карыстальніку.
Як я ўжо казаў, рэдагаванне рэалізуецца праз форму, такім чынам, нам патрэбны
две формы: для дзеянняў *new* і *edit*. Але навошта рабіць две аднолькавыя формы,
калі мы умеем ствараць частковыя шаблоны?

```ruby
#app/views/students/_form.html.erb
<%= form_for @student, html: {class: 'form-horizontal center'} do |f| %>

    <% if @student.errors.any? %>
        <div class="alert alert-danger">
          <b><%= pluralize(@student.errors.count, "error") %> on form.</b>
          <ul>
            <% @student.errors.full_messages.each do |message| %>
                <li>* <%= message %></li>
            <% end %>
          </ul>
        </div>
    <% end %>

    <div class="form-group">
      <%= f.label :name, class: 'col-md-4 control-label' %>
      <div class="col-md-6">
        <%= f.text_field :name, class: 'form-control' %>
      </div>
    </div>

    <div class="form-group">
      <%= f.label :github_user, class: 'col-md-4 control-label' %>
      <div class="col-md-6">
        <%= f.text_area :github_user, class: 'form-control' %>
      </div>
    </div>

    <%= f.submit "#{action_name} student", class: 'btn btn-large btn-primary' %>
<% end %>
```

Тут мы выкарысталі інтэрпаляцыю для таго, каб адлюстраваць назву кнопкі адпаедную
дзеянню `<%= f.submit "#{action_name} student", class: 'btn btn-large btn-primary' %>`.

Засталося толькі адрэндэрыць гэтую форму на views *new* і *edit*:

```ruby
#app/views/students/new.html.erb
<div class="page-header">
  <h1>New student</h1>
</div>

<%= render 'form'%>
```

```ruby
#app/views/students/new.html.erb
<div class="page-header">
  <h1>Edit student</h1>
</div>

<%= render 'form'%>
```

Зараз можна абнавіць навігацыю і прыступіць да запаўнення базы студэнтаў
непасрэдна праз web-інтэрфэйс.

``` ruby
  <ul class="nav navbar-nav navbar-left">
    <li><%= link_to 'Home', root_path %></li>
    <li><%= link_to 'Course', course_path %></li>
    <li><%= link_to 'Testimonials', testimonials_path %></li>
    <li><%= link_to 'Students', students_path %></li>
  </ul>
```
Не забываем абнавіць Хероку:

    $ git push heroku master

Такім чынам, у нас ужо ёсць паўнавартаснае web-прыкладанне з сістэмай
аўтэнтыфікацыі і адміністрацыйнай часткай. Наша прыкладанне разгорнута на
воблачным серверы і узаемадзейнічае з базай дадзеных. Пагадзіцеся, ужо някепскі
вынік, улічваючы прастату з якой мы гэта зрабілі. На астатніх занятках мы пашырым
функцыянал прыкладання і зробім яго больш інтэрактыўным.


[<< папярэдні занятак](7_lecture.md)
[наступны занятак >>](9_lecture.md)
