# Nested Form

Conveniently manage multiple nested models in a single form. It does so in an unobtrusive way through jQuery.

This gem works with Rails 3+.

## Setup

Add it to your Gemfile then run `bundle` to install it.

```ruby
gem 'nested_form'
```

And then add it to the Asset Pipeline in `app/assets/javascripts/application.js` file:

```
//= require jquery_nested_form
```

### Non Asset Pipeline Setup

If you do not use the asset pipeline, run this generator to create the JavaScript file.

```bash
rails g nested_form:install
```

You can then include the generated JavaScript in your layout.

```erb
<%= javascript_include_tag :defaults, "nested_form" %>
```

### Rails 5+ and Webpacker

As Nested Form is a very old project, support for a NPM package was never provided and although being old enough, this gem works fine with newer versions of Rails up until 5.2.3.

If you're using [webpacker](https://github.com/rails/webpacker) and want to introduce Nested Form to your project, you can use the NPM verion that is just an entrypoint to the file located in `vendor/assets/javascripts/jquery_nested_form`.

> You can manually download the file and import it as well but I wanted to try publishing it to NPM.

Add it to your project with:

```bash
$ yarn add jquery_nested_form
```

And import it in your `application.js` pack:

```javascript
import 'jquery_nested_form'
```

## Usage

- [Strong Parameters](#strong-parameters)
- [SimpleForm and Formtastic Support](#simpleform-and-formtastic-support)
- [Partials](#partials)
- [Specifying target when adding Nested Fields](#specifying-target-when-adding-nested-fields)
- [Specifying wrapper to remove Nested Fields](#specifying-wrapper-to-remove-nested-fields)

Imagine you have a `Project` model that `has_many :tasks`. To be able to use this gem, you'll need to add `accepts_nested_attributes_for :tasks` to your Project model. If you wish to allow the nested objects to be destroyed, then add the `:allow_destroy => true` option to that declaration.

> See the [accepts_nested_attributes_for documentation](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html#method-i-accepts_nested_attributes_for) for details on all available options.

This will create a `tasks_attributes=` method, so you may need to add it to the `attr_accessible` array (`attr_accessible :tasks_attributes`).

Then use the `nested_form_for` helper method to enable the nesting.

```erb
<%= nested_form_for @project do |f| %>
```

You will then be able to use `link_to_add` and `link_to_remove` helper methods on the form builder in combination with fields_for to dynamically add/remove nested records.

```erb
<%= f.fields_for :tasks do |task_form| %>
  <%= task_form.text_field :name %>
  <%= task_form.link_to_remove "Remove this task" %>
<% end %>
<p><%= f.link_to_add "Add a task", :tasks %></p>
```

In order to choose how to handle, after validation errors, fields that are
marked for destruction, the `marked_for_destruction` class is added on the div
if the object is marked for destruction.

### Strong Parameters

For Rails 4+ here is an example:

```ruby
params.require(:project).permit(:name, tasks_attributes: [:id, :name, :_destroy])
```

The `:id` is to make sure you do not end up with a whole lot of tasks.

The `:_destroy` must be there so that we can delete tasks.

### SimpleForm and Formtastic Support

Use `simple_nested_form_for` or `semantic_nested_form_for` for SimpleForm and Formtastic support respectively.

### Partials

It is often desirable to move the nested fields into a partial to keep things organized. If you don't supply a block to fields_for it will look for a partial and use that.

```erb
<%= f.fields_for :tasks %>
```

In this case it will look for a partial called "task_fields" and pass the form builder as an `f` variable to it.

### Specifying target when adding Nested Fields

By default, `link_to_add` appends fields immediately before the link when
clicked. This is not desirable when using a list or table, for example. In
these situations, the "data-target" attribute can be used to specify where new
fields should be inserted.

```erb
<table id="tasks">
  <%= f.fields_for :tasks, :wrapper => false do |task_form| %>
    <tr class="fields">
      <td><%= task_form.text_field :name %></td>
      <td><%= task_form.link_to_remove "Remove this task" %></td>
    </tr>
  <% end %>
</table>
<p><%= f.link_to_add "Add a task", :tasks, :data => { :target => "#tasks" } %></p>
```

### Specifying wrapper to remove Nested Fields

By default, `link_to_remove` works by hiding all fields wrapped in div element with `.fields` class.

Having this wrapper is completely optional and can be disabled with `wrapper: false`. However, when disabling the wrapper, `link_to_remove` stops working because it needs to somehow reference the DOM element that holds all fields to "remove".

When using `wrapper: false`, normally one would introduce a default wrapper. Maybe because of a normal wrapper like a bootstrap `.row`.

In those cases, the custom wrapper **class name** should be specify in the `link_to_remove` helper:

```erb
<%= f.link_to_remove 'Quitar', data: { wrapper: '.row' } %>
```

> Notice the use of CSS dot notation for class name `.row`

## JavaScript events

Sometimes you want to do some additional work after element was added or removed, but only after DOM was _really_ modified. In this case simply listening for click events on
'Add new'/'Remove' link won't reliably work, because your code and code that inserts/removes nested field will run concurrently.

This problem can be solved, because after adding or removing the field a set of custom events is triggered on this field. Using form example from above, if you click on the "Add a task" link, `nested:fieldAdded` and `nested:fieldAdded:tasks` will be triggered, while `nested:fieldRemoved` and `nested:fieldRemoved:tasks` will be triggered if you click "Remove this task" then.

These events bubble up the DOM tree, going through `form` element, until they reach the `document`. This allows you to listen for the event and trigger some action accordingly. Field element, upon which action was made, is passed along with the `event` object. In jQuery you can access it via `event.field`.

For example, you have a date input in a nested field and you want to use jQuery datepicker for it. This is a bit tricky, because you have to activate datepicker after field was inserted.

See example below for jQuery:

```javascript
$(document).on('nested:fieldAdded', function(event){
  // this field was just inserted into your form
  var field = event.field; 
  // it's a jQuery object already! Now you can find date input
  var dateField = field.find('.date');
  // and activate datepicker on it
  dateField.datepicker();
})
```

Second type of event (i.e. `nested:fieldAdded:tasks`) is useful then you have more than one type of nested fields on a form (i.e. tasks and milestones) and want to distinguish, which exactly was added/deleted.

> See also [how to limit max count of nested fields](https://github.com/ryanb/nested_form/wiki/How-to:-limit-max-count-of-nested-fields)

## Enhanced jQuery JavaScript template

You can override default behavior of inserting new subforms into your form. For example:

```javascript
window.nestedFormEvents.insertFields = function(content, assoc, link) {
  return $(link).closest('form').find(assoc + '_fields').append($(content));
}
```

## Contributing

If you have any issues with Nested Form not addressed above or in the [example project](https://github.com/ryanb/complex-form-examples/tree/nested_form), please add an [issue on GitHub](https://github.com/ryanb/nested_form/issues) or [fork the project](https://help.github.com/articles/fork-a-repo) and send a [pull request](https://help.github.com/articles/using-pull-requests). To run the specs:

```
bundle install
bundle exec rake spec:install
bundle exec rake db:migrate
bundle exec rake spec:all
```

See available rake tasks using `bundle exec rake -T`.

## Special Thanks

This gem was originally based on the solution by Tim Riley in his [complex-form-examples fork](https://github.com/timriley/complex-form-examples/tree/unobtrusive-jquery-deep-fix2).

Thank you Andrew Manshin for the Rails 3 transition, [Andrea Singh](https://github.com/madebydna) for converting to a gem and [Peter Giacomo Lombardo](https://github.com/pglombardo) for Prototype support.

Andrea also wrote a great [blog post](http://blog.madebydna.com/all/code/2010/10/07/dynamic-nested-froms-with-the-nested-form-gem.html) on the internal workings of this gem.

Thanks [Pavel Forkert](https://github.com/fxposter) for the SimpleForm and Formtastic support.
