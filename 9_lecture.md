# 9. Асацыяцыі Active Record

Паставім новую задачу. Дадзім магчымасць выкладчыку-карыстальніку нашага
прыкладання весці некалькі курсаў.  У сваю чаргу, кожны з курсаў будзе мець
пэўных наведвальнікаў. Rails дазваляе усталёўваць сувязі (асацыяцыі) паміж
мадэлямі. Цягам занятка, мы разбярэмся, як гэта працуе.

Пачнем са стварэння мадэлі Course з атрыбутам *name*:

    $ rails g model course name:string --no-test-framework

Створым адпаведны кантролер:

    $ rails g controller courses index show  --no-helper --no-assets --no-test-framework

Маршрут статычнай старонкі *course* i саму старонку
**app/views/pages/course.html.erb** зараз можна выдаліць. Згенераваныя маршруты
для курсаў заменім на больш звыклыя рэсурсныя:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root  'pages#home'

  get 'mentor' => 'mentor#index'

  controller :sessions do
    get  'login' => :new
    post 'login' => :create
    delete 'logout' => :destroy
  end

  resources :testimonials, only: [:index, :new, :show, :create, :destroy]
  resources :courses, only: [:index, :show]

  resources :students
end
```

Не забываем абнавіць адпаведную спасылку у навігацыі:

    # app/views/shared/_header.html.erb
    <li><%= link_to 'Courses', courses_path %></li>

Для таго, каб стварыць узаемасувязь (асацыяцыю) паміж курсамі і студэнтамі на
узроўне дадзеных, неабходна дадаць у табліцу *students* слупок з ідэнтыфікатарамі курсаў:

    $ rails g migration AddCourseToStudent course:belongs_to

Такім чынам, кожны студэнт можа быць прывязаны да пэўнага курса. У сваю чаргу,
кожны курс можа мець шмат студэнтаў. Гэта усяго толькі адзін з варыянтаў
рэалізацыі сувязяў. Усе залежыць ад патрабаванняў да дадзеных: напрыклад, можна
будзе змяніць сувязі такім чынам, каб студэнт мог наведваць некалькі курсаў.

Зараз трэба прапісаць гэтыя залежнасці у мадэлях. Пазначым у мадэлі Course, што
курс можа мець шмат студэнтаў. Заадно дададзім валідацыі:

```ruby
# app/models/course.rb
class Course < ActiveRecord::Base
  has_many :students
  validates :name, presence: true, uniqueness: true
end
```

І зваротная залежнасць. Кожны студэнт належыць курсу:

    # app/models/student.rb
    belongs_to :course

Давайце дададзім яшчэ адзін слупок да табліцы студэнтаў, у якім будзем захоўваць
імя heroku-прыкладання:

    $ rails generate migration AddHerokuAppToStudents heroku_app:string

Не забываем дадаць адпаведнае поле у форму студэнта:

```ruby
# app/views/students/_form.html.erb
    <div class="form-group">
      <%= f.label 'Heroku app name', class: 'col-md-4 control-label' %>
      <div class="col-md-6">
        <%= f.text_field :heroku_app, class: 'form-control' %>
      </div>
    </div>
```

Каб паглядзець на справе, як працуюць асацыяцыі, давайце запоўнім табліцу
*students* дадзенымі. Мы зробім гэта праз *seeds* файл, які зможам змяняць у
будучыні у выпадку мадыфікацыі структуры дадзеных. На пачатковых этапах распрацоўкі
гэта добрая практыка, з дапамогай якой можна сэканоміць шмат часу і пазбегнуць
"малпавай" працы. Таксама дадамо дадзеныя для курса, каб не марнаваць час на
стварэнне формы і адпаведных дзеянняў для кантроллера (вы ужо можаце зрабіць гэта самі).

```ruby
# db/seeds.rb
# drop courses and students
puts 'Drop students...'
Student.delete_all
puts 'Drop courses...'
Course.delete_all

# create courses
puts 'Creating courses...'
courses = Course.create!([
                             { name: 'Building web applications with Ruby on Rails' },
                             { name: 'HTML & CSS basics' }
                         ])

# create students
puts 'Creating students...'
students = [
    {
        github_user: 'alesdrobysh',
        name: 'Ales',
        course: courses[0],
        heroku_app: 'sleepy-lowlands-6650'
    },
    {
        github_user: 'qwerchenok',
        name: 'Alexey',
        course: courses[0],
        heroku_app: 'satdatabase'
    },
    #...
    {
        github_user: 'alberteinstein',
        name: 'Albert',
        course: courses[1],
        heroku_app: nil
    }
]

students.map { |s| Student.create!(s) }
```

Заўважце, што спачатку мы выдаляем ўсе запісы студэнтаў і курсаў, а потым ствараем
новыя запісы. Таксама паглядзіце, як перадаецца асацыяцыя з курсам падчас стварэння
запіса студэнта. Ну і пазнаёмцеся з метадам [map](http://ruby-doc.org/core-2.2.0/Array.html#method-i-map).

Каб запоўніць БД дадзенымі seed-файла, трэба выканаць каманду

    $ rake db:seed

Дадаем стандартныя дзеянні у кантролер курсаў:

```ruby
# app/controllers/courses_controller.rb
class CoursesController < ApplicationController
  def index
    @courses = Course.all
  end

  def show
    @course = Course.find(params[:id])
  end
end

Дадаем старонку са спісам курсаў:

```ruby
# app/views/courses/index.html.erb
<div class="page-header">
  <h1>Courses</h1>
</div>

<div class="list-group">
  <%= render @courses %>
</div>
```

І адпаведны частковы шаблон:

```ruby
# app/views/courses/_course.html.erb
<%= link_to course.name, course_path(course), class: 'list-group-item' %>
```

Зараз давайце абнавім старонку студэнта, дададзім спасылкі на GitHub і на Heroku.
Для таго, каб не перагружаць шаблон, створым адмысловыя метады-хэлперы для спасылак:

```ruby
# app/helpers/students_helper.rb
module StudentsHelper

  def github_link(student)
    link_to 'GitHub account', "https://github.com/#{student.github_user}",
            target: '_blank', class: 'list-group-item'
  end

  def heroku_link(student)
    link_to 'Heroku application', "https://#{student.heroku_app}.herokuapp.com/",
            target: '_blank', class: 'list-group-item' if student.heroku_app
  end
end
```
Выносіць логіку з прадстаўленняў - добрая практыка. Хэлперы вельмі добра
падыходзяць для гэтага, асабліва калі гэтыя метады выкарыстоўваюцца у розных файлах view.

Зараз можна выкарыстаць гэтыя метады на шаблоне студэнта:

```ruby
# app/views/students/show.html.erb
<div class="page-header list-group">
  <h2><%= @student.name %></h2>
  <%= github_link(@student) %>
  <%= heroku_link(@student) %>
</div>

<% if session[:mentor_id] %>
    <%= link_to "edit", edit_student_path(@student), class: 'btn btn-default' %>
    <%= link_to 'delete', @testimonial, method: :delete, class: 'btn btn-danger',
                data: { confirm: "You sure?" } %>
<% end %>
```


Заўважце, што спасылка на хероку дадаецца толькі у тым выпадку, калі ёсць значэнне *heroku_app*.

Дзякуючы асацыяцыям Active Record, мы можам атрымаць спіс студэнтаў курса. Rails
стварае для нас асацыятыўныя метады для маніпуляцыі дадзенымі, напрыклад:
`course.students` - гэты метад верне усіх студэнтаў курса. Альбо `student.course`
- верне курс студэнта. Альбо `course.students.count` - падлік запісаў студэнтаў
у БД з адпаведным *course_id*. Прычым гэтыя метады робяць вельмі разумныя запыты
у БД, напрыклад, калі мы падлічваем студэнтаў пэўнага курса, падлік робіцца
непасрэдна ў БД, без неабходнасці выцягваць УСІХ студэнтаў. Падлічваюцца толькі
студэнты з адпаведным *course_id*. На мове SQL гэта выглядае прыкладна так:

    SELECT COUNT(*) FROM "students" WHERE "students"."course_id" = $1  [["course_id", 4]]

Заўсёды трымайце у галаве, што кожны запыт да БД робіць нагрузку на выша
прыкладанне і узаемадзеянне з БД - гэта тонкае у перформансе (прадукцыйнасці)
вэб-прыкладання.

Перададзім на старонку кожнага курса студэнтаў адпаведнага курса і колькасць
студэнтаў гэтага курса. Дзякуючы асацыятыўным метадам і ўжо гатоваму частковаму
шаблону студэнта гэта задача робіцца супер-простай:

```ruby
# app/views/course/show.html.erb
<h1><%= @course.name %></h1>
<p><span class="badge">Students count: <%= @course.students.count %></span></p>
<%= render @course.students%>
```

Разгледзім больш складаную задачу з сувязямі. Звяжам студэнтаў з выкладчыкам праз
курсы. Для гэтага выкарыстаем тыпы сувязі *has_many through* і *has_one through*.
Зараз разбярэмся, як гэта выглядае на узроўні мадэляў.

Выкладчык можа весці некалькі курсаў, у сваю чаргу, у выкладчыка ёсць студэнты,
якія ходзяць на гэтыя курсы:

```ruby
# app/models/mentor.rb
  has_many :courses
  has_many :students, through: :courses
```

Студэнт ходзіць на курс, у сваю чаргу гэты курс вядзе выкладчык, таму у студэнта
ёсць адзін выкладчык:

```ruby
# app/models/student.rb
  belongs_to :course
  has_one :mentor, through: :course
```


У курса можа быць шмат студэнтаў і адзін выкладчык:

```ruby
# app/models/course.rb
  has_many :students
  belongs_to :mentor
```

Мы звязалі две мадэлі (ментар і студэнт) праз трэцюю мадэль (курс).

Для таго, каб асацыятыўныя метады працавалі, трэба дадаць міграцыю, якая дадасць
слупок з *id* выкладчыка у табліцу курсаў:

    $ rails g migration AddMentorToCourse mentor:references

Тып *references* - гэта тое ж самае, што *belongs_to*. Заўважце, што табліцу
студэнтаў мы не чапаем. Сувязь студэнт-выкладчык мы рэалізавалі на узроўні
мадэляў і праз табліцу *courses*.

Зараз трэба дадаць сувязь курсаў з выкладчыкам у базе дадзеных, зробім гэта праз
seed-file. Трэба абнавіць кавалак кода, які стварае курсы:

```ruby
# create courses
puts 'Creating courses...'
mentor = Mentor.find_by_name('micrum')
courses = Course.create!([
                             { name: 'Building web applications with Ruby on Rails', mentor: mentor},
                             { name: 'HTML & CSS basics', mentor: mentor }
                         ])
```

Абнавім кантролер:

```ruby
# app/controllers/mentor_controller.rb
  def index
    @mentor = Mentor.find_by(id: session[:mentor_id])
    @courses = @mentor.courses
  end
```

Абнавім адмін-старонку, дададзім інфамацыю пра курсы і студэнтаў:

```ruby
# app/views/mentor/index.html.erb
<div class="page-header">
  <h1>Welcome Admin!</h1>
</div>

<h2>Current courses:</h2>
<div class="list-group">
  <%= render @courses %>
</div>

<h2>Total students count: <%= @mentor.students.count%></h2>
```

Звярніце увагу, якім добрым было рашэнне загадзя стварыць частковы шаблон
**app/views/courses/_course.html.erb**. І паглядзіце, якую добрую працу робяць
для нас асацыяцыі Active Record. Мы на *index* старонцы Ментара можам атрымаць
дадзеныя мадэлі Студэнта без умяшальніцтва у кантролер. Я ужо не кажу пра тое,
насколькі легкімі і лексічна простымі з'яўляюцца асацыятыўныя метады, якія даюць
вялікія магчымасці для працы з дадзенымі у вашым Rails application.

Падрабязней пра асацыяцыі чытайце у
[гайдах](http://guides.rubyonrails.org/association_basics.html). Гэта апошняя
фундаментальная тэма нашага курса. Зараз вы ўжо здольныя будаваць паўнавартасныя
дынамічныя web-прыкладанні з дапамогай Rails. Дык працуйце!


[<< папярэдні занятак](8_lecture.md)
[наступны занятак >>](10_lecture.md)
