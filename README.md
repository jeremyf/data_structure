# DataStructure

Programatically declare your data structures for various contexts.

Why DataStructure?
We are asked to model a breadth of topics: from Books, to Datasets, to Brains.
And for those topics, we are asked to render things in specific and rather arbitrary roles: editor, author, admin.
And for those topics, we also need to render things in a specific mode: discovery, show, edit.

## Installation

Add this line to your application's Gemfile:

    gem 'data_structure'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install data_structure

## Usage

Bear with me as these are aspirations and entirely subject to change.
Ultimately, I want to model the data structure of an object in its various contexts.
I also want to be mindful that we could auto-build a data structure from a well-formed underlying model.

```ruby
# With externally declared data structure
class Book < SomeOrm
  class Structure
    include DataStructure::Container
    attribute :title, mode: [:edit, :show], role: :editor, section: :required, as: 'Attribute::Title'
    attribute :administrative_notes, mode: [:show], role: :editor, section: :optional

    role :editor do
      validate :title, presence: true
    end
    mode :new do
      validate :title, uniquness: true
    end

  end

  attr_accessor :title
  attr_accessor :description

  def to_data_structure(context, options)
    Book::Structure.new(self, context, options)
  end
end

# With inline declared data structure
class Book < SomeOrm
  include DataStructure
  data_structure do
    attribute :title, mode: [:edit, :show], role: :editor, section: :required, as: Attribute::Title
    attribute :description, mode: [:show], role: :editor, section: :optional

    role :editor do
      validate :title, presence: true
    end
    mode :new do
      validate :title, uniquness: true
    end

  end
end
```

The above two methods could be construed as logically equivalent.

```ruby
# A mean of easily declaring a Value Object
# By wrapping a Literal with an Object we can programatically differentiate
# what the Form and Show element that we may want to render.
class DataStructure::Attribute
  def initialize(context, name)
    @context = context
    @name = name
  end
  delegate :model, :mode, :role, to: :context
end
```

```ruby
# app/controllers/books_controller.rb
class BooksController < ApplicationController

  def show
    book = Book.find(params[:id])
    @book = book.to_data_structure(self, mode: :show, role: current_user_role)
  end

  def edit
    book = Book.find(params[:id])
    @book = book.to_data_structure(self, mode: :edit, role: current_user_role)
  end

  private
  def current_user_role
    current_user.is_admin? ? :editor : nil
  end
end
```

```erb
# app/views/book/edit.html.erb
<%= form_for(@book, builder: DataStructure::FormBuilder) do |f| %>
  <% @book.each_attribute_for_section(:required) do |attribute| %>
    <!-- In this case we would have the :f context as we are using the above builder -->
    <%= attribute.render %>
  <% end %>
<% end %>

# app/views/book/show.html.erb
<%= show_for(@book, builder: DataStructure::ShowBuilder) do |s| %>
  <% @book.each_attribute_for_section(:required) do |attribute| %>
    <!-- In this case we would have the :s context as we are using the above builder -->
    <%= attribute.render %>
  <% end %>
<% end %>
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
