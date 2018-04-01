<h1>TDD Workshop</h1>

<h2>Goal</h2>

<p>
In this workshop, we'll help you make your first steps with Test Driven Development in Rails, using RSpec. RSpec is a widely used testing framework that can be used un pure ruby and also
have helpers for Rails and other web developement frameworks.
</p>

<p>
We will start with an already existing application just to get started and learn about how to write a test, the different types of tests, good practices. Then, once we added some different tests,
we'll develop a new feature using Test Driven Development.
</p>

<h2>Install rvm</h2>

<a href=https://rvm.io/>https://rvm.io/</a><br>

<tt>\curl -sSL https://get.rvm.io | bash -s stable</tt>


<h2>Clone one of the repos at:</h2>

<tt>git clone -b tdd-workshop https://github.com/bluciam/railsbridge-montreal-website.git</tt>

<h2>Get started</h2>

Testing in Rails is both hard and easy. Hard because it takes as much time as coding does, but easy because there are many tools that make the most complicated things straightforward. There are so many tools though, that sometimes it is hard to choose. Neat paradox.

When setting up testing for my app, I did not find in one place a comprehensive explanation of the Rails testing process as a whole. This is an attempt to fix the situation.
<h2>Why testing?</h2>
The issue is more what and when to test. Michael Hartl has an introduction to testing in his <a title="Ruby on Rails tutorial" href="https://www.railstutorial.org/book/static_pages#sec-getting_started_with_testing" target="_blank" rel="noopener">Ruby on Rails tutorial</a>, which describes why testing and the kinds of tests he is likely to do. In summary,
<ul>
	<li>first test the controllers and the models, in other words <code>unit testing</code>;</li>
	<li>second test the functionality across models, views and controllers, the <code>integration tests</code>;</li>
	<li>and third, the views, but if they are likely to change, they can be skipped.</li>
</ul>
He also mentions the importance of writing <code>regression tests</code> on bugs found and having tests in place before any refactoring.
<h2>RSpec testing infrastructure</h2>
<h3>Set up</h3>
When a new Rails project is created with default settings, a <code>test</code> directory is created, coupled to work with <code>minitest</code>. From the word GO, I started using <code>RSpec</code> (<a title="RSpec home page" href="http://rspec.info/" target="_blank" rel="noopener">http://rspec.info/</a>, since I used the <em>Ruby on Rails tutorial</em> mentioned above which used <code>RSpec</code> as testing framework. I believed it has changed since.

I did not want that default directory, as I wanted the RSpec set up.

By issuing the command
<pre>rails g controller StaticPages about --no-test-framework</pre>
the test files related to the <code>StaticPages controller</code> will not be created. To create the right files, RSpec must be installed. That is done by including its gem in the Gemfile. <code>RSpec</code> takes advantage of a series of helpers to run tests automatically. The gems are specified following:
<pre>group :development, :test do
  gem 'rspec-rails'
  gem 'factory_bot_rails'
end

group :test do
  gem 'selenium-webdriver'
  gem 'faker'
  gem 'capybara'
  gem 'guard-rspec'
  gem 'launchy'
end
</pre>
and then run bundle.

In my opinion, the most interesting about Rails testing, is the interaction with databases and <code>RSpec</code> plus helpers makes this easy.
<blockquote>By default, every Rails application has three environments: development, test, and production. The database for each one of them is configured in <code>config/database.yml</code>.

<a title="Ruby on Rails guides" href="http://guides.rubyonrails.org/testing.html" target="_blank" rel="noopener">http://guides.rubyonrails.org/testing.html</a></blockquote>
That fact is simply brilliant, as the development database is NOT the same as the test database. One can create automatically hundreds of records to test for specific features in isolation.

To run any testing, the databases need to be created, so run
<pre>rake db:create:all</pre>
which will create the databases which do not exist and inform you of the ones already created. The information on the databases description is taken from the file <code>config/database.yml</code>, which should include the information about test, development and production databases. Make sure that the three are name differently!

Then run
<pre>rails generate rspec:install</pre>
which generates the following configuration files:
<pre>.rspec
spec/spec_helper.rb
spec/rails_helper.rb
</pre>
All tests and helpers will reside in the <code>spec directory</code>. This is directory where <code>RSpec</code> searches for the tests to run.

NOTE: After making changes to any of the models in development, you have to migrate the changes to the test database as well by running
<pre>rake db:migrate RAILS_ENV=test</pre>
With this, the testing infrastructure is set up.
<h3>Syntax</h3>
<a href="http://www.rubydoc.info/github/rspec/rspec-core/frames" target="_blank" rel="noopener">RSpec</a> uses mainly the words "describe" and "it" so we can express concepts like a conversation:
<blockquote>"Describe an order."
"It sums the prices of its line items."</blockquote>
The <code>describe</code> method creates an <code>ExampleGroup</code>. Within the block passed to <code>describe</code> you can declare examples using the <code>it</code> method. Under the hood, an example group is a class in which the block passed to <code>describe</code> is evaluated.

The broad syntax of the test is as follows:
<pre>describe <em>Object</em> do
  it "Descriptive message of the test" do
    <em>code with expectations</em>
  end
end
</pre>
Each "it" line only expects one example. Best practice is to test one thing at a time to make it simple to find errors. Although the descriptive message is technically optional, omitting defeat the purpose of individual testing. Previous <code>RSpec</code> examples had the "should" beginning the message, however that just clutters the output. A direct verb suffices.
<h3>Actual testing</h3>
Now to writing the tests. But where to start? With how to run a test.

To run a test, use the command
<pre>rspec</pre>
from the root directory of the app. If used alone, it will run all the tests found in the <code>spec</code> directory. You can also specify a directory or a filename including its path with respect to root. RSpec will run all tests found in that directory in the first case, or just the file specified in the second.

The testing framework automatically creates directories to sort out the tests. My <code>spec</code> directory looks like this:
<pre>controllers/  factories/       models/         requests/       support/
helpers/      rails_helper.rb  spec_helper.rb  views/
</pre>
Next is what to test: unit testing, integrations testing, views, regression testing.
<h3>Unit testing: Models</h3>
<blockquote>Models are the building blocks of the application. They are also easier to test since their behaviour should be well defined in any application. I considered them to be first priority to test.

<a title="Blog Everyday Rails" href="http://everydayrails.com/2012/03/19/testing-series-rspec-models-factory-girl.html" target="_blank" rel="noopener">Everyday Rails</a></blockquote>
The blog <a title="Blog Everyday Rails" href="http://everydayrails.com/2012/03/19/testing-series-rspec-models-factory-girl.html" target="_blank" rel="noopener">Everyday Rails</a> considers the following to be essential model tests:
<ul>
	<li>the <code>factory</code> should generate a valid object</li>
	<li>data validation</li>
	<li>class and instance method</li>
</ul>
Full text deployed in a post called <em><a title="Testing with RSpec in Rails, Part 2, Models" href="https://superspreadsheet.wordpress.com/2015/01/27/testing-with-rspec-in-rails-part-2-models/" target="_blank" rel="noopener">Testing with RSpec in Rails, Part 2, Models</a></em>.
<h3>Unit testing: Controllers</h3>
Post in development.
<h3>Integration tests</h3>
Post in conception.
This should test the functionality across models, views and controllers.
<h3>View's tests?</h3>
Will I create a post?
Code in the views tend to change often, so there are different schools of thought on whether to test or not.
<h3>Regression tests</h3>
<h2>Automatic test runs with Guard</h2>
<code>Guard</code> runs the test suite upon the detection of a modification of file in the spec directory or as specified in the <code>Guardfile</code>. It also sets the testing environment just once, speeding up the running of subsequent tests. To set up (gem already included in the Gemfile) run:
<pre>bundle exec guard init
</pre>
which creates the <code>Guardfile</code> describing how and when Guard is to run. In his tutorial, <a title="Guard: Michael Hartl's Rails tutorial" href="https://www.railstutorial.org/book/static_pages#sec-guard" target="_blank" rel="noopener">Guard: Michael Hartl's Rails tutorial</a>, he explains the set up in more detail, although using <code>minitest</code> instead of <code>RSpec</code>.

To start guard just type in a terminal
<pre>$ guard</pre>
It will create a shell and <code>guard</code> will start listening to any changes in the spec directory or any other file specified in the <code>Guardfile</code>. The <code>Guarfile</code> created in the set up is a very good starting point.

Type enter in the shell to run all the tests in the spec directory. <code>Ctl-D</code> to exit.

(Explanation on <code>Guard</code> needs expansion.)


<em>(Edited 26 February 2015, added information about accessing database details.)</em>

Following from the post <a href="https://superspreadsheet.wordpress.com/2015/01/22/testing-with-rspec-in-rails/" title="Testing with RSpec in Rails, Part 1, Introduction" target="_blank">Testing with RSpec in Rails, Part 1, Introduction</a>, it is now time to expand on testing models based on my experience.

I considered three types of test relevant for my application:
<ul>
	<li>Factory tests</li>
	<li>Data fields validations</li>
	<li>Associations between models</li>
</ul>

<h2>Factory test</h2>

The first test is to make sure that a valid record can be created safely and respecting all of the constraints. In other words, that is has a valid <em>factory</em>.

<blockquote>A factory is an object for creating other objects – formally a factory is simply an object that returns an object from some method call, which is assumed to be "new".

<a href="https://en.wikipedia.org/wiki/Factory_%28object-oriented_programming%29#cite_ref-1" title="Factory definition, wikipedia" target="_blank">https://en.wikipedia.org/wiki/Factory_%28object-oriented_programming%29#cite_ref-1</a>
</blockquote>

In my opinion, this is not a TDD type test, rather, it follows the database design. Therefore, the table might exist already. This is what I am assuming for the rest of this post.

If the table exists, we need tools to find the table structure in the development database. There are two easy ways that I know of using the rails command. The first is using the console <pre>rails c</pre> short for <pre>rails console</pre>
The command
<pre>
$ rails c
Loading development environment (Rails 4.0.4)
2.0.0-p451 :001 > ActiveRecord::Base.connection.tables
 => ["schema_migrations", "users", ...]
</pre>
will output an array including all of the current tables. And the command <code>User.column_names</code>

<pre>
2.0.0-p451 :008 > User.column_names
 => ["id", "name", "email", ...]
</pre>
will output an array including the column names of the table User.

The second way of accessing this information is by using the command
<pre>rails db</pre>
short for
<pre>rails dbconsole</pre>
It starts a console for the database and database adapter specified in <code>config/database.yml</code> depending on the current Rails environment. If testing, one is most likely in the development environment.
<pre>
$ rails db
Password:
psql (9.4.1, server 9.3.5)
Type "help" for help.

App_Name_development=#
</pre>
To retrieve the fields of the <em>articles</em> table, I used the following command:
<pre>=# select column_name from information_schema.columns where table_name='articles';</pre>

The output is:
<pre>
 column_name
-------------
 id
 name
 bio
 image
 created_at
 updated_at
(6 rows)
</pre>

The command
<pre>
App_Name_developemnt=# \d
</pre>
will display a list of all the relations.

With this information and with the information in the <code>model.rb</code> file, the factory can be written for the model.

I used the <code>faker</code> gem to create random names and descriptions (<a href="https://github.com/stympy/faker" title="The Faker gem, github" target="_blank">https://github.com/stympy/faker</a>). Given that <em>authors</em> write <em>articles</em>, I also have a factory for articles. The factories look like this:
<pre>
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
</pre>

The <code>image</code> field has been created with the <code>carrierwave</code> gem and that is the code needed in the testing environment. The file <code>image_2.jpg</code> must exist in the <code>spec/support/images/</code> directory.

I did not know what was best practice between having one file called <code>spec/factories.rb</code> or having a file <code>model_name.rb</code> in the <code>spec/factories</code> directory. My searches did not deliver any conclusive results. So I used trial and error. I first opted for one file, then a mix, and then multiple files in the directory, as the number of factories grew, one per model. This is in-line with Rails practice of always having small files.

The actual test to validate the factory is written in file <code>spec/models/article_spec.rb</code>:
<pre>
require 'rails_helper'

describe Article do
  # Validation tests
  it "has a valid factory" do
    expect(FactoryGirl.create(:article)).to be_valid
  end
end
</pre>

The RSpec matcher <code>be_valid</code> verifies that our factory does indeed return a valid object.

Given that I had already created my models, once the test was passing, I changed the code to see the test fail, making sure that specific parts of code were indeed being tested.

<h2>Data validations</h2>

These tests are straight-forward, validating any of the constraints needed in each field. The  model of the article has the constraint that the title is mandatory, as shown below:
<pre>
# app/models/article.rb
class Article < ActiveRecord::Base
  validates :title, presence: true
...
</pre>
The following lines, create a object in the test environment, and gives it an empty title, which is not allowed:

<pre>
require 'rails_helper'

describe Article do
...
  it "is invalid without a title" do
    expect(FactoryGirl.build(:article, title: nil)).not_to be_valid
  end
...
end
</pre>

Notice in the <code>article_spec.rb</code> file there are two special methods: <code>create</code> and <code>build</code>.
<code>create</code> builds and saves the object, while <code>build</code> only does that. This allows the modification of attributes before saving the object. The <code>not_to</code> verifies that an empty title should not be allowed in a valid object.


<h2>Associations between models</h2>

An interesting test that I discovered in this <a href="http://liahhansen.com/2011/04/14/testing-model-associations-in-rspec.html" title="Testing associations in RSpec" target="_blank">post</a>, is testing the associations between models using RSpec. As a beginner, I always want to double check that I made the right associations. For this, I used the gem <code>shoulda</code> found in <a href="https://github.com/thoughtbot/shoulda" title="shoulda gem" target="_blank">github</a>.

Add the gem to the Gemfile:
<pre>
group :test do
...
  gem 'shoulda-matchers'
end
</pre>
and run <code>bundle</code>.

Then add to the <code>spec/models/article_spec.rb</code>, the following test:
<pre>
require 'rails_helper'

describe Article do
...

  # Associations test
  it { should belong_to(:author) }
end
</pre>

