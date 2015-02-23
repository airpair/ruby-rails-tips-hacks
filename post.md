## 1 Introduction

Ruby on Rails is a great full-stack framework to build web applications. It is very opinionated, and provides abstraction to web development. The conventions and features of Rails allow developers to focus on design and development, without having to worry about the inner workings of the web server. 

Rails, which is currently at version 4.2, has come very far. The community-driven development has helped Rails to mature as a framework and addressed many needs of web developers. Although there have been numerous new frameworks introduced recently,  I consider Rails to be an all-in-one solution for web development which makes it superior to others.

Familiarity with the framework improves development speed, efficiency, code maintainability and architecture. It takes a significant amount of time to learn everything that Rails has to offer and to master the framework. I have compiled a list of hacks and tips that I use in my day to day development, and I hope that these pointers will help you to become an effective Rails developer. 


## 2 Rails environment setup

RVM is a popular Ruby manager, which simplifies the installation and management of Rubies. RVM is pretty simple to install. You can follow the instruction at [RVM.io](http://rvm.io/). 

### 2.1 Manage Ruby (RVM)

Almost all developers work with multiple Rails projects on the same machine. They can be client projects, side projects or experiments. Each of those projects may use a different version of Ruby.

It is difficult to keep track of the project-specific Ruby versions when switching between projects. However, there is a solution: all you have to do is create a `.ruby-version` file in the root of your Rails project and add the following line:

<!--code lang=ruby linenums=true-->

    ruby-2.1.2
Whenever you navigate to the project folder from your terminal, RVM will switch to the correct Ruby. Of course, the version has to correspond to the version of your choice. In our case it is 2.1.2.

If you want to know what rubies are installed on your machine, you can run the following command:

<!--code lang=bash linenums=true-->

    $  blog_app  rvm list
    rvm rubies

       rbx-2.2.10 [ x86_64 ]
       rbx-2.2.9 [ x86_64 ]
       ruby-2.0.0-p481 [ x86_64 ]
     * ruby-2.1.1 [ x86_64 ]
    => ruby-2.1.2 [ x86_64 ]

    # => - current
    # =* - current && default
    #  * - default

Use the string (e.g. rbx-2.2.10, ruby-2.1.2, etc.) to set the correct Ruby version/implementation.

In case the version was not found in the installed Rubies, you will get the following error:

<!--code lang=bash linenums=true-->

    $  cd blog_app
    Unknown ruby string (do not know how to handle): ruby-2.1.0.
    ruby-2.1.0 is not installed.
    To install do: 'rvm install ruby-2.1.0'

This ensures that anyone who is working on your project will use the correct Ruby.

### 2.2 Manage Gemsets (RVM)

RVM provides Gemsets to isolate the Ruby environments. You can take advantage of that by creating a Gemset per project per Ruby version. 
In the `.ruby-version` file, in addition to setting the Ruby version, you need to add the Gemset name like so:

<!--code lang=ruby linenums=true-->

    ruby-2.1.2@blog_app

Then navigate to your project directory in your terminal; your new Gemset will be created as follows:

<!--code lang=bash linenums=true-->

    $ cd blog_app
    ruby-2.1.2 - #gemset created /Users/backnol/.rvm/gems/ruby-2.1.2@blog_app
    ruby-2.1.2 - #generating blog_app wrappers - please wait

## 3 Rails console

Rails console is a powerful tool. It is a playground for developers to test things out. It comes in handy when you want to try out something new, such as a new gem that you just installed. The more you know about the console, more you can wield its power. 

### 3.1 Helper methods

View helpers are used in Rails to keep your code DRY. Rails comes with many view helpers (e.g. link_to, number_to_currency, etc.). In addition to that you can create your own view helpers under the `app/helpers` directory. All the view helpers are available and can be used in the views. If you wish to try out these helpers in the console without having to refresh the browser, you can do so with the `helper` object. The helper methods are available as instance methods of this object.

<!--code lang=ruby-->

    >> helper.number_to_currency(10)
     => "$10.00"
    >> helper.number_to_percentage(22, precision: 2)
     => "22.00%"

### 3.2 Sandbox mode

When you play around in the console, any actions that you perform are executed against the development environment. Database changes as a result of the model calls can be persisted. If you prefer to try things out without inadvertently changing the data, you can start the console in sandbox mode by running:

<!--code lang=bash linenums=true-->

    $ rails c --sandbox 

All the `.create`, `.update`,  `.save` and other database-related actions that you do will not be persisted. The operations that you perform in this mode will only be effective for that session only. Once you quit the console, all your changes will be lost. The sandbox mode is truly a no strings attached way of testing things out. 

### 3.3 Access last evaluated result

Imagine that you ran `Post.find(1)` to get a particular post in the console. However you forgot to store the object in a variable. The underscore comes to the rescue. The result of the expression that you last ran will be stored in the `\_` variable:

<!--code lang=ruby-->

    >> Post.find(2)
      Post Load (0.3ms)  SELECT  "posts".* FROM "posts"  WHERE "posts"."id" = $1 LIMIT 1  [["id", 2]]
     => #<Post id: 2, title: "Ruby", body: "Rails is awesome", created_at: "2014-10-02 03:14:10", updated_at: "2014-10-02 03:14:10">
    >> _
     => #<Post id: 2, title: "Ruby", body: "Rails is awesome", created_at: "2014-10-02 03:14:10", updated_at: "2014-10-02 03:14:10">
    >> post = _
     => #<Post id: 2, title: "Ruby", body: "Rails is awesome", created_at: "2014-10-02 03:14:10", updated_at: "2014-10-02 03:14:10">

### 3.4 Reload! not restart

Console comes with a `reload!` command to reload your classes without restarting. However, this does not reload the setup, initializers and libraries. 

<!--code lang=ruby-->

    >> reload!
    Reloading...
     => true

## 4 ActiveRecord

ActiveRecord is the heart of Rails. It helps the developer to execute SQL queries on the database without even writing a single line of SQL. 

### 4.1 Query by date range

ActiveRecord provides a simple way to query objects for a date range. The following example is a query to find the posts created between 10 days ago and now: 

<!--code lang=ruby-->

    >> Post.where(created_at: 10.days.ago..Time.now).to_sql
     => "SELECT \"posts\".* FROM \"posts\"  WHERE (\"posts\".\"created_at\" BETWEEN '2014-09-22 02:30:12.021169' AND '2014-10-02 02:30:12.021500')"

### 4.2 Conditional associations

Conditional associations come in handy when you want to access a subset of the children.

<!--code lang=ruby linenums=true-->

    class User
      has_many :posts
      has_many :authorized_posts, ->{ where(approved: true) }, class_name: 'Post'
    end

    User.find(1).authorized_posts

This will return posts that are approved by the admin.

## 5 Command line

Rails comes packed with some commands that you can execute in the the terminal in your project directory. The tools such as `generate` and `rake` are designed to speed up the development process.

### 5.1 Pretend generate

The `rails generate` command will be used many times during the lifetime of a project (i.e migration, scaffold, rspec:install, etc.). Sometimes the generate may create or modify multiple files that you may not want to. If you need to find out what files will be created/modified, you can run a pretend migration:

<!--code lang=ruby linenums=true-->

    $ blog_app  rails g model comment body:text post_id:integer -p
          invoke  active_record
          create    db/migrate/20141002023923_create_comments.rb
          create    app/models/comment.rb
          invoke    test_unit
          create      test/models/comment_test.rb
          create      test/fixtures/comments.yml

### 5.2 rake notes

We developers often work under tight deadlines, which may prevent us from implementing some "good to have" things. In such case we add comments that begins with:

<!--code lang=ruby linenums=true-->

    # TODO
    # FIXME
    # OPTIMIZE

You can do a search for the string within your project to see all these comments. Alternatively, Rails provide `rake notes`, which will show the list of the comments that begins with these strings:

<!--code lang=ruby linenums=true-->

    $  rake notes
    app/controllers/application_controller.rb:
      * [6] [FIXME] Title is not dynamic for each page

    app/controllers/posts_controller.rb:
      * [7] [TODO] It is better if we can avoid these unecessary joins.

    app/models/post.rb:
      * [3] [OPTIMIZE] This is too slow. Better to use pluck

## 6 Performance
Performance affects the user experience in production environments and development speed in development environments. It is always good to run your code fast.

### 6.1 Automatically clear logs

Log files are used to track the actions that happen in the server. If you do not pay attention, you may lose precious space on your hard drive to the log files. You can clear the log by running:

<!--code lang=ruby linenums=true-->

    rake log:clear

You can automate it during server startup by adding this snippet to the initializer:

<!--code lang=ruby linenums=true-->

    # config/initializers/clear_logs.rb
    if Rails.env.development?
      MAX_LOG_SIZE = 2.megabytes

      logs = File.join(Rails.root, 'log', '*.log')
      if Dir[logs].any? {|log| File.size?(log).to_i > MAX_LOG_SIZE }
        $stdout.puts "Runing rake log:clear"
        `rake log:clear`
      end
    end

Keep your logs clean, save space, and have peace of mind.

### 6.2 Benchmark your code

Controller is too slow? Model is too slow? Query is too slow? Rails comes with `Benchmark.ms` that allows you to evaluate the time taken to execute a block of code.

<!--code lang=ruby linenums=true-->

    time = Benchmark.ms{ Post.all }
    puts time

You can use `Benchmark.ms` with your code within the braces to evaluate the time taken for execution.

## 7 Conclusion

Ruby on Rails is a powerful framework that comes with all the bells and whistles to write web applications. Learning all what it has to offer will require a lot of experience. The list of tips and hacks that I have provided will cover some important aspects to master the framework.