# 10. Выкарыстанне JavaScript у Ruby on Rails прыкладаннях. AJAX. Калбэкі Rails

JavaScript - мова праграмавання, якая можа выконваецца у браўзеры. Менавіта гэта 
асаблівасць робіць JavaScript самай ужыванай МП у свеце. У вэб-прыкладаннях 
JavaScript можа выкарыстоўвацца для наступных мэтаў:

- кліентская частка вэб-прыкладння (інтэрактыўны інтэрфэйс і г.д)
- бэкэнд (NodeJs)
- AJAX

## AJAX

У кантэксце Rails-прыкладанняў у першую чаргу нас цікавіць AJAX (asynchronous 
JavaScript and XML) і трошкі інтэрактыунасць.

Давайце зірнем, што адбываецца, калі мы адпраўляем водгук на сайт нашага прыкладання. 
Спачатку мы націскаем кнопку "Add testimonial" на старонцы index, далей на новай 
старонцы атрымліваем форму для водгука, адпраўляем дадзеныя з формай і у выпадку 
паспяховага стварэння водгука атрымліваем зноў index-старонку. Занадта складана, ці не праўда? 

Замест таго, каб на кожны запыт атрымліваць ад сервера адказ у выглядзе новай 
старонкі, мы можам абнаўляць толькі частку старонкі. Справа у тым, што запыты і 
апрацоўку адказаў можна рабіць не толькі сродкамі Rails, але і з дапамогай 
JavaScript. Нам не заўсёды трэба атрымліваць усе дадзеныя з сервера, часта 
дастаткова атрымаць толькі частку і на аснове новых дадзеных абнавіць частку 
старонкі. Гэта і ёсць AJAX.

Зараз мы пераробім працэс дадання водгукаў. Мы будзем паказваць форму стварэння 
водгука у мадальным акне на старонцы index. І пасля стварэння дададзім новы 
водгук на старонку.

Стварыць мадальнае акно нам дапаможа Bootstrap (трэба толькі падключыць JS):

    # app/assets/javascripts/application.js
    //= require bootstrap-sprockets

Таксама спатрэбяцца bootstrap-cтылі для кампанентаў:

    # app/assets/stylesheets/styles.scss
    @import "bootstrap-sprockets";

Пераробім форму водгука, змясцім у мадальнае акно адпаведна структуры 
[Bootstrap](http://getbootstrap.com/javascript/#modals):

```ruby
# app/views/testimonials/new.html.erb
<div class="modal fade" id="newTestimonialModal" tabindex="-1" role="dialog" 
     aria-labelledby="myModalLabel" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-body">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>

        <%= form_for :testimonial, remote: true, url: testimonials_path, 
                     html: {class: 'form-horizontal center testimonial-form'} do |f| %>

            <div class="form-group">
              <%= f.label :user_name, class: 'col-md-4 control-label' %>
              <div class="col-md-6">
                <%= f.text_field :user_name, class: 'form-control' %>
              </div>
            </div>

            <div class="form-group">
              <%= f.label :feedback, class: 'col-md-4 control-label' %>
              <div class="col-md-6">
                <%= f.text_area :feedback, class: 'form-control' %>
              </div>
            </div>

            <%= f.submit 'Create testimonial', class: 'btn btn-large btn-primary' %>
        <% end %>
      </div>
    </div>
  </div>
</div>
```

Для таго, каб рэалізаваць AJAX у форме, дастаткова было дадаць код `remote: true` у метад `form_for`.

Змесцім форму у частковы шаблон:

    $ mv app/views/testimonials/new.html.erb app/views/testimonials/_new_testimonial.html.erb

Зараз мы можам дадаць мадальны дыялог з формай на старонку index:

```ruby
# app/views/testimonials/index.html.erb
<div class="page-header">
    <h1>Testimonials</h1>
</div>

<div class="panel panel-default">
  <div class="panel-body">
    <%= link_to 'Add new testimonial', '#', class: 'btn btn-primary btn-lg',
                data: {toggle: 'modal', target: '#newTestimonialModal'} %>
  </div>
</div>

<div class="list-group">
  <%= render @testimonials %>
</div>

<%= render 'new_testimonial'%>
```

Маршрут `:new` нам больш не патрэбны: стварэнне водгука будзе адбывацца на index старонцы:

    # config/routes.rb
    resources :testimonials, only: [:index, :show, :create, :destroy]

Такім чынам, дзякуючы `remote: true`, форма будзе апрацоўвацца JavaScript. 
Засталося толькі зрабіць так, каб Testimonials кантролер адказваў на AJAX-запыты 
і абнавіць дзеянні. Для гэтага скарыстаемся гемам [responders](https://github.com/plataformatec/responders):

    # Gemfile
    gem 'responders', '~> 2.0'

Абнаўляем кантролер. Мы рэалізуем AJAX праз прыем і перадачу JSON-дадзеных:

```ruby
# app/controllers/testimonials_controller.rb
class TestimonialsController < ApplicationController

  respond_to :json

  before_action :authorize, only: :destroy

  def index
    @testimonials = Testimonial.order("id DESC").all
  end

  def show
    @testimonial = Testimonial.find(params[:id])
  end

  def create
    @testimonial = Testimonial.create(testimonial_params)
    respond_with @testimonial
  end

  def destroy
    Testimonial.find(params[:id]).destroy
    redirect_to testimonials_url
  end

  private

  def testimonial_params
    params.require(:testimonial).permit(:user_name, :feedback)
  end
end
```

Мы выдалілі дзеянне **new()**, яно больш не патрэбна, бо форму мы паказваем на 
index-старонцы. Таксама мы абнавілі **index()** дзеянне - водгукі адлюстроўваліся 
у парадку стварэння, зараз зверху старонкі мы будзем паказваць самыя новыя водгукі. 
Таксама мы спрасцілі **create()** дзеянне. Больш не трэба рабіць перанакіраванне 
і паказваць новую старонку. Нам трэба толькі атрымаць json-адказ ад сервера.

Падрабязней пра рэалізацыю REST API з дапамогай метада **respond_to** чытайце 
[тут](http://blog.plataformatec.com.br/2009/08/embracing-rest-with-mind-body-and-soul/).

## Coffeescript

Мы выдалілі дзеянне **new()** і код з праверкай памылак. Але важна паказваць 
памылкі карыстальніку, зараз мы зробім гэта з дапамогай JavaScript (дакладней 
[CoffeeScript](http://coffeescript.org/) - гэта лаканічная мова, якая кампілюецца у JavaScript).

```coffeescript
# app/assets/javascript/testimonials.coffee
$ ->
  $modal = $('#newTestimonialModal')

  $('.testimonial-form').on("ajax:success", (e, status) ->
    $(@)[0].reset()
    $modal.modal 'hide'    
  ).on "ajax:error", (e, status) ->
    errors = status.responseJSON.errors
    for key of errors
      $("#testimonial_#{key}").closest('.form-group').addClass('has-error')

  $modal.on('hide.bs.modal', (e) ->
    $('.form-group').removeClass('has-error')
  )
```

Мы таксама выкарысталі [jQuery](https://jquery.com/) - бібліятэку, якая спрашчае 
аперацыі з [DOM](http://www.w3schools.com/js/js_htmldom.asp). Давайце разглядзім, 
што робіць гэты код. Дзякуючы магчымасцям jQuery мы можам "павесіць" на элементы 
функцыі-апрацоўшчыкі пэўных падзеяў (events), такіх як *'ajax:error'* (памылка 
падчас адпраўкі формы) альбо *'hide.bs.modal'* (закрыццё мадальнага акна).

Мы апрацавалі паспяховае стварэнне водгука а таксама выпадак з памылкамі. Спачатку 
мы правяраем, ці прыходзіць паспяховы адказ на ajax-запыт з формы водгука? Калі 
так - вычышчаем інпуты формы і хаваем мадальнае акно. Калі не - атрымліваем 
JSON-паведамленне з памылкамі і надаем клас 'has-error' адпаведнаму інпуту. Не 
забываем пра тое, што клас з памылкамі трэба зняць з інпутаў пасля таго, як зачынілі форму.

Засталося толькі зрабіць, каб водгук дадаваўся на старонку без перазагрузкі старонкі. 
Для гэтага можна выкарыстаць embedded Ruby і метад `render`, каб дадаць частковы 
шаблон з водгукам. Але такі падыход прадугледжвае змешванне логікі прыкладання. 
Можаце паглядзець прыклад [тут](https://coderwall.com/p/kqb3xq/rails-4-how-to-partials-ajax-dead-easy).

Замест гэтага мы будзем дадаваць новыя водгукі на старонку з дапамогай JavaScript, 
заадно папрактыкуемся. Але спачатку крыху абнавім частковы шаблон водгука, каб 
было зручней магіпуляваць HTML элементамі.

```ruby
# app/views/testimonials/_testimonial.html.erb
<a href="<%= testimonial_path(testimonial) %>" class="list-group-item testimonial">
  <blockquote>
    <p class="testimonial-feedback"><%= testimonial.feedback %></p>
    <footer class="testimonial-footer">
      <%= testimonial.user_name %>
      <span class="testimonial-time"><%= time_ago_in_words(testimonial.created_at) %> ago</span>
    </footer>
  </blockquote>
</a>
```

Зараз зоймемся абнаўленнем старонкі. У выніку паспяховага стварэння водгука, 
выклічам функцыю `prependNewTestimonial(feedback, user_name, id)`, якая 
прычэпіць водгук на старонку. У параметры гэтай функцыі мы перадаем тэкст 
новага водгука, імя карыстальніка і ідэнтыфікатар. Значэнні гэтых параметраў мы 
атрымаем з адказа ад сервера і пакуль толькі пакажам іх у JavaScript-кансолі.

```coffeescript
# app/assets/javascript/testimonials.coffee
$ ->
  $modal = $('#newTestimonialModal')

  $('.testimonial-form').on("ajax:success", (e, status) ->
    $(@)[0].reset()
    $modal.modal 'hide'
    feedback = status.feedback
    user_name = status.user_name
    id = status.id
    prependNewTestimonial(feedback, user_name, id)
  ).on "ajax:error", (e, status) ->
    errors = status.responseJSON.errors
    for key of errors
      $("#testimonial_#{key}").closest('.form-group').addClass('has-error')

  $modal.on('hide.bs.modal', (e) ->
    $('.form-group').removeClass('has-error')
  )

  prependNewTestimonial = (feedback, user_name, id) ->
    console.log("feedback: #{feedback}", user name: #{user_name}, id: #{id}")
```

Засталося толькі напісаць саму функцыю. 

```coffeescript
# app/assets/javascript/testimonials.coffee
  prependNewTestimonial = (feedback, user_name, id) ->
    $last_testimonial = $('.testimonial').first().clone().prependTo('.list-group')
    url = $last_testimonial.attr('href')
    last_testimonial_id = url.substring(url.lastIndexOf('/') + 1)
    new_url = url.replace(last_testimonial_id, id)
    $last_testimonial.attr('href', new_url)
    $last_testimonial.find('.testimonial-feedback').text(feedback)
    $last_testimonial.find('.testimonial-footer').text(user_name + ' Just now')
```

Спачатку мы "кланіравалі" апошні (верхні) водгук. Затым замянілі спасылку 
апошняга водгука на спасылку на новы водгук, пры гэтым мы скарысталі параметр *id*. 
Таксама замянілі тэкст водгука "клона" і імя карыстальніка.

## Калбэкі Rails

Давайце зробім невялікую цэнзуру у водгуках. Заменім якое-небудзь непрыгожае 
слова на больш пазітыўнае. Для гэтага выкарыстаем 
[зваротныя выклікі Rails](http://guides.rubyonrails.org/active_record_callbacks.html).

Жыццёвы цыкл аб'екта ў Rails - гэта стварэнне, абнаўленне і выдаленне. Rails 
дазваляе умяшацца у гэты жыццёвы цыкл і пераключыць логіку перад/пасля альбо 
падчас змянення/стварэння/выдалення аб'екта.

Паглядзіце як гэта працуе на практыцы:

```ruby
# app/models/testimonial.rb
class Testimonial < ActiveRecord::Base
  validates :user_name, :feedback, presence: true
  before_save :censorship

  private

  def censorship
    if feedback =~ /\bshit\b/
      self.feedback = feedback.downcase.gsub! /\bshit\b/, 'fun'
    end
  end
end
```

Мы выкарысталі метад **gsub!**, каб замяніць у радку усе словы "shit" на "fun" 
у тэксце водгука. Метад з клічнікам у Ruby змяняе сам аб'ект, на якім выкліканы 
метад. Перад гэтым выклікалі метад **downcase()**, каб "Shit" і іншыя спалучэнні 
кшталту "sHit" таксама не прайшлі цэнзуру. Апроч гэтага у дадзеным прыкладзе мы 
сутыкнуліся з [рэгулярным  выразам](http://ruby-doc.org/core-2.2.0/Regexp.html): 
праз `\b` пазначаем пачатак і канец слова.

[<< папярэдні занятак](9_lecture.md)
[наступны занятак >>](11_lecture.md)
