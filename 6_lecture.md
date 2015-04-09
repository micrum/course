# 6. Уводзіны у аб'ектна-арыентаванае праграмаванне. Асновы мовы Ruby

New testimonial form

    # app/views/testimonials/new.html.erb
    <%= form_for @testimonial, html: {class: 'form-horizontal center'} do |f| %>

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

Array with errors

    $ rails c
    > testimonial = Testimonial.new
    > testimonial.save
    > errors_array = testimonial.errors.full_messages
    > errors_array[0]

Display errors on form

    # app/views/testimonials/new.html.erb
    <% if @testimonial.errors.any? %>
      <div class="alert alert-danger">
        <b><%= pluralize(@testimonial.errors.count, "error") %> on form.</b>
        <ul>
          <% @testimonial.errors.full_messages.each do |message| %>
              <li>* <%= messge %></li>
          <% end %>
        </ul>
      </div>
    <% end %>

Errors styles

    #app/stylesheets/custom.scss

    .field_with_errors {
      input, textarea {
      border-color: red;
      }

      label {
        color: red;
      }
    }

Index action

    # app/controllers/testimonials_controller.rb
    def index
      @testimonials = Testimonial.all
    end

Index view

    # app/views/testimonials/index.html.erb
    <div class="panel panel-default">
      <div class="panel-body">
        <%= link_to 'Add new testimonial', new_testimonial_path, class: 'btn btn-primary btn-lg' %>
      </div>
    </div>

    <div class="list-group">
      <%= render @testimonials %>
    </div>

Testimonial partial

    # app/views/testimonial/_testimonial.html.erb
    <a href="<%= testimonial_path(testimonial) %>" class="list-group-item">
      <blockquote>
        <p><%= testimonial.feedback %></p>
        <footer>
          <%= testimonial.user_name %>
          <span><%= time_ago_in_words(testimonial.created_at) %> ago</span>
        </footer>
      </blockquote>
    </a>

[<< папярэдні занятак](5_lecture.md)
[наступны занятак >>](7_lecture.md)