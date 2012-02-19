# Sequenced

Sequenced is a simple Rails 3 engine that generates scoped sequential 
IDs for ActiveRecord models. The sequential ID does not replace the database
primary key, but rather adds another way to retrieve the object within the 
scope of another attribute.

Extracted from the [GuideKit](https://guidekit.com) codebase.

## Purpose

It's generally a bad practice to expose your primary keys to the world 
in your URLs. However, often times it is appropriate to number objects in
sequence in the context of a parent object.

For example, given a Question model that has many Answers, it makes sense
to number answers sequentially for each individual question. You can achieve 
this with Sequenced in one line of code:

```ruby
class Question < ActiveRecord::Base
  has_many :answers
end

class Answer < ActiveRecord::Base
  belongs_to :question
  acts_as_sequenced :scope => :question_id
end
```

## Installation

First, add the gem to your Gemfile:

```    
gem 'sequenced'
```
    
Then, install and run migrations:

```
rails generate sequenced
rake db:migrate
```

## Usage

To add a sequential ID to a model, first add an integer column called
`sequential_id` to the model (or you many name the column anything you
like and override the default). For example:

```
rails generate migration add_sequential_id_to_answers sequential_id:integer
rake db:migrate
```

Then, call the `acts_as_sequenced` macro in your model class:

```ruby
class Answer < ActiveRecord::Base
  belongs_to :question
  acts_as_sequenced :scope => :question_id
end
```

The `:scope` option can be any attribute, but will typically be the foreign
key of an associated parent object.

## Configuration

### Overriding the default sequential ID column

By default, Sequenced uses the `sequential_id` column and assumes it already 
exists. If you wish to store the sequential ID in different integer column, 
simply specify the column name with the `:column` option:

```ruby
acts_as_sequenced :scope => :question_id, :column => :my_sequential_id
```

### Starting the sequence at a specific number

By default, Sequenced begins sequences with 1. To start at a different 
integer, simply set the `:start_at` option:

```ruby
acts_as_sequenced :scope => :question_id, :start_at => 1000
```

## Example

Suppose you have a question model that has many answers. This example 
demonstrates how to use Sequenced to enable access to a resource
via its sequential ID.

```ruby
# app/models/question.rb
class Question < ActiveRecord::Base
  has_many :answers
end

# app/models/answer.rb
class Answer < ActiveRecord::Base
  belongs_to :question
  acts_as_sequenced :scope => :question_id
  
  # Automatically use the sequential ID in URLs
  def to_param
    self.sequential_id
  end
end

# config/routes.rb
resources :questions
  resources :answers
end

# app/controllers/answers_controller.rb
class AnswersController < ApplicationController
  before_filter :load_question
  before_filter :load_answer, :only => [:show, :edit, :update, :destroy]
  
private

  def load_question
    @question = Question.find(params[:question_id])
  end
  
  def load_answer
    @answer = @question.answers.where(:sequential_id => params[:id]).first
  end
end
```

Now, answers are accessible via their sequential IDs:

    http://example.com/questions/5/answers/1

## License

Copyright &copy; 2012 Derrick Reimer

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.