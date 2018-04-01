# Goal

In this workshop, we'll help you make your first steps with Test Driven Development in Rails, using RSpec. RSpec is a widely used testing framework that can be used un pure ruby and also have helpers for Rails and other web developement frameworks.

We will start with an already existing application just to get started and learn about how to write a test, the different types of tests, good practices. Then, once we added some different tests, we'll develop a new feature using Test Driven Development.

# Install rvm

```
\curl -sSL https://get.rvm.io | bash -s stable
```

**RVM** is a Ruby version manager. There are many others, like **rbenv** and **asdf**. It is useful when you have multiple Ruby applications on your computer, using different ruby versions. You don't have to install **RVM** if you already have a version manager.

# Clone one of the repos at:

```
git clone -b tdd-workshop https://github.com/bluciam/railsbridge-montreal-website.git
cd railsbridge-montreal-website
bundle install
```

# Get started

Testing in Rails is both hard and easy. Hard because it takes as much time as coding does, but easy because there are many tools that make the most complicated things straightforward. There are so many tools though, that sometimes it is hard to choose. Neat paradox. When setting up testing for my app, I did not find in one place a comprehensive explanation of the Rails testing process as a whole. This is an attempt to fix the situation.

# Why testing?

The issue is more what and when to test. Michael Hartl has an introduction to testing in his [Ruby on Rails tutorial](https://www.railstutorial.org/book/static_pages#sec-getting_started_with_testing "Ruby on Rails tutorial"), which describes why testing and the kinds of tests he is likely to do. In summary,

*   first test the controllers and the models, in other words `unit testing`;
*   second test the functionality across models, views and controllers, the `integration tests`;
*   and third, the views, but if they are likely to change, they can be skipped.

He also mentions the importance of writing `regression tests` on bugs found and having tests in place before any refactoring.

# RSpec testing infrastructure

## Set up

When a new Rails project is created with default settings, a `test` directory is created, coupled to work with `minitest`. From the word GO, I started using `RSpec` ([http://rspec.info/](http://rspec.info/ "RSpec home page"), since I used the _Ruby on Rails tutorial_ mentioned above which used `RSpec` as testing framework. I believed it has changed since. I did not want that default directory, as I wanted the RSpec set up. By issuing the command

```
rails g controller StaticPages about --no-test-framework

the test files related to the `StaticPages controller` will not be created. To create the right files, RSpec must be installed. That is done by including its gem in the Gemfile. `RSpec` takes advantage of a series of helpers to run tests automatically. The gems are specified following:

group :development, :test do
  gem 'rspec-rails'
  gem 'factory\_bot\_rails'
end

group :test do
  gem 'selenium-webdriver'
  gem 'faker'
  gem 'capybara'
  gem 'guard-rspec'
  gem 'launchy'
end

```

and then run bundle. In my opinion, the most interesting about Rails testing, is the interaction with databases and `RSpec` plus helpers makes this easy.

> By default, every Rails application has three environments: development, test, and production. The database for each one of them is configured in `config/database.yml`. [http://guides.rubyonrails.org/testing.html](http://guides.rubyonrails.org/testing.html "Ruby on Rails guides")

That fact is simply brilliant, as the development database is NOT the same as the test database. One can create automatically hundreds of records to test for specific features in isolation. To run any testing, the databases need to be created, so run

```
rake db:create:all
```

which will create the databases which do not exist and inform you of the ones already created. The information on the databases description is taken from the file `config/database.yml`, which should include the information about test, development and production databases. Make sure that the three are name differently! Then run

```
rails generate rspec:install
```

which generates the following configuration files:

```
.rspec
spec/spec_helper.rb
spec/rails_helper.rb
```

All tests and helpers will reside in the `spec directory`. This is directory where `RSpec` searches for the tests to run. NOTE: After making changes to any of the models in development, you have to migrate the changes to the test database as well by running

```
rake db:migrate RAILS_ENV=test
```

With this, the testing infrastructure is set up.

## Syntax

[RSpec](http://www.rubydoc.info/github/rspec/rspec-core/frames) uses mainly the words "describe" and "it" so we can express concepts like a conversation:

> "Describe an order." "It sums the prices of its line items."

The `describe` method creates an `ExampleGroup`. Within the block passed to `describe` you can declare examples using the `it` method. Under the hood, an example group is a class in which the block passed to `describe` is evaluated. The broad syntax of the test is as follows:

```
describe _Object_ do
  it "Descriptive message of the test" do
    _code with expectations_
  end
end

```

Each `it` line only expects one example. Best practice is to test one thing at a time to make it simple to find errors. Although the descriptive message is technically optional, omitting defeat the purpose of individual testing. Previous `RSpec` examples had the "should" beginning the message, however that just clutters the output. A direct verb suffices.

## Actual testing

Now to writing the tests. But where to start? With how to run a test. To run a test, use the command

```
rspec
```

from the root directory of the app. If used alone, it will run all the tests found in the `spec` directory. You can also specify a directory or a filename including its path with respect to root. RSpec will run all tests found in that directory in the first case, or just the file specified in the second. The testing framework automatically creates directories to sort out the tests. My `spec` directory looks like this:

```
controllers/  factories/       models/         requests/       support/
helpers/      rails\_helper.rb  spec\_helper.rb  views/
```

Next is what to test: unit testing, integrations testing, views, regression testing.

## Unit testing: Models

> Models are the building blocks of the application. They are also easier to test since their behaviour should be well defined in any application. I considered them to be first priority to test. [Everyday Rails](http://everydayrails.com/2012/03/19/testing-series-rspec-models-factory-girl.html "Blog Everyday Rails")

The blog [Everyday Rails](http://everydayrails.com/2012/03/19/testing-series-rspec-models-factory-girl.html "Blog Everyday Rails") considers the following to be essential model tests:

*   the `factory` should generate a valid object
*   data validation
*   class and instance method

Full text deployed in a post called _[Testing with RSpec in Rails, Part 2, Models](https://superspreadsheet.wordpress.com/2015/01/27/testing-with-rspec-in-rails-part-2-models/ "Testing with RSpec in Rails, Part 2, Models")_.

## Unit testing: Controllers

Post in development.

## Integration tests

Post in conception. This should test the functionality across models, views and controllers.

## View's tests?

Will I create a post? Code in the views tend to change often, so there are different schools of thought on whether to test or not.

## Regression tests

# Automatic test runs with Guard

`Guard` runs the test suite upon the detection of a modification of file in the spec directory or as specified in the `Guardfile`. It also sets the testing environment just once, speeding up the running of subsequent tests. To set up (gem already included in the Gemfile) run:

bundle exec guard init

which creates the `Guardfile` describing how and when Guard is to run. In his tutorial, [Guard: Michael Hartl's Rails tutorial](https://www.railstutorial.org/book/static_pages#sec-guard "Guard: Michael Hartl's Rails tutorial"), he explains the set up in more detail, although using `minitest` instead of `RSpec`. To start guard just type in a terminal

```
guard
```

It will create a shell and `guard` will start listening to any changes in the spec directory or any other file specified in the `Guardfile`. The `Guarfile` created in the set up is a very good starting point. Type enter in the shell to run all the tests in the spec directory. `Ctl-D` to exit. (Explanation on `Guard` needs expansion.) _(Edited 26 February 2015, added information about accessing database details.)_ Following from the post [Testing with RSpec in Rails, Part 1, Introduction](https://superspreadsheet.wordpress.com/2015/01/22/testing-with-rspec-in-rails/ "Testing with RSpec in Rails, Part 1, Introduction"), it is now time to expand on testing models based on my experience. I considered three types of test relevant for my application:

*   Factory tests
*   Data fields validations
*   Associations between models

# Factory test

The first test is to make sure that a valid record can be created safely and respecting all of the constraints. In other words, that is has a valid _factory_.

> A factory is an object for creating other objects – formally a factory is simply an object that returns an object from some method call, which is assumed to be "new". [https://en.wikipedia.org/wiki/Factory_%28object-oriented\_programming%29#cite\_ref-1](https://en.wikipedia.org/wiki/Factory_%28object-oriented_programming%29#cite_ref-1 "Factory definition, wikipedia")

In my opinion, this is not a TDD type test, rather, it follows the database design. Therefore, the table might exist already. This is what I am assuming for the rest of this post. If the table exists, we need tools to find the table structure in the development database. There are two easy ways that I know of using the rails command. The first is using the console

```
rails c # short for rails console
```

The command

```
$ rails c
Loading development environment (Rails 4.0.4)
2.0.0-p451 :001 > ActiveRecord::Base.connection.tables
 =\> \["schema_migrations", "users", ...\]
```

will output an array including all of the current tables. And the command `User.column_names`

```
2.0.0-p451 :008 > User.column_names
 =\> \["id", "name", "email", ...\]
```

will output an array including the column names of the table User. The second way of accessing this information is by using the command

```
rails db # short for rails dbconsole
```

It starts a console for the database and database adapter specified in `config/database.yml` depending on the current Rails environment. If testing, one is most likely in the development environment.

```
$ rails db
Password:
psql (9.4.1, server 9.3.5)
Type "help" for help.

App\_Name\_development=#

```

To retrieve the fields of the _articles_ table, I used the following command:

```
=# select column\_name from information\_schema.columns where table_name='articles';
```

The output is:

```
 column_name
\-\-\-\-\-\-\-\-\-\-\-\-\-
 id
 name
 bio
 image
 created_at
 updated_at
(6 rows)

```

The command

App\_Name\_developemnt=# \\d

will display a list of all the relations. With this information and with the information in the `model.rb` file, the factory can be written for the model. I used the `faker` gem to create random names and descriptions ([https://github.com/stympy/faker](https://github.com/stympy/faker "The Faker gem, github")). Given that _authors_ write _articles_, I also have a factory for articles. The factories look like this:

```Ruby
# spec/factories.rb
require 'faker'

FactoryGirl.define do

  factory :author do
    name Faker::Name.name
    bio { Faker::Lorem.sentences.to_s }
    image { Rack::Test::UploadedFile.new(File.join(Rails.root, 'spec',
      'support', 'images', 'image_2.jpg')) }
  end

  factory :article do
    author
    title "The life of Pepito Perez"
    summary { Faker::Lorem.sentence }
  end
end

```

The `image` field has been created with the `carrierwave` gem and that is the code needed in the testing environment. The file `image_2.jpg` must exist in the `spec/support/images/` directory. I did not know what was best practice between having one file called `spec/factories.rb` or having a file `model_name.rb` in the `spec/factories` directory. My searches did not deliver any conclusive results. So I used trial and error. I first opted for one file, then a mix, and then multiple files in the directory, as the number of factories grew, one per model. This is in-line with Rails practice of always having small files. The actual test to validate the factory is written in file `spec/models/article_spec.rb`:

```Ruby
require 'rails_helper'

describe Article do
  # Validation tests
  it "has a valid factory" do
    expect(FactoryGirl.create(:article)).to be_valid
  end
end

```

The RSpec matcher `be_valid` verifies that our factory does indeed return a valid object. Given that I had already created my models, once the test was passing, I changed the code to see the test fail, making sure that specific parts of code were indeed being tested.

# Data validations

These tests are straight-forward, validating any of the constraints needed in each field. The model of the article has the constraint that the title is mandatory, as shown below:

```Ruby
# app/models/article.rb
class Article < ActiveRecord::Base
  validates :title, presence: true
```


The following lines, create a object in the test environment, and gives it an empty title, which is not allowed:

```Ruby
require 'rails_helper'

describe Article do

  it "is invalid without a title" do
    expect(FactoryGirl.build(:article, title: nil)).not\_to be\_valid
  end

end

```

Notice in the `article_spec.rb` file there are two special methods: `create` and `build`. `create` builds and saves the object, while `build` only does that. This allows the modification of attributes before saving the object. The `not_to` verifies that an empty title should not be allowed in a valid object.

# Associations between models

An interesting test that I discovered in this [post](http://liahhansen.com/2011/04/14/testing-model-associations-in-rspec.html "Testing associations in RSpec"), is testing the associations between models using RSpec. As a beginner, I always want to double check that I made the right associations. For this, I used the gem `shoulda` found in [github](https://github.com/thoughtbot/shoulda "shoulda gem"). Add the gem to the Gemfile:

```ruby
group :test do
  gem 'shoulda-matchers'
end

```

and run `bundle`. Then add to the `spec/models/article_spec.rb`, the following test:

```ruby
require 'rails_helper'

describe Article do

  # Associations test
  it { should belong_to(:author) }
end
```

