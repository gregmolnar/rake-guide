# Rails and rake

As I already mentioned Rails uses Rake for various tasks like running the database migrations. Today we are going to look at these tasks.
If you enter rake -T you will get a list of Rake tasks but this only displays the ones with a description, so the ones intended to be internal are not on this list. If you want to see those too you can use the -A switch:

```
$ rake -T -A
rake about                      # List versions of all Rai...
rake assets:clean[keep]         # Remove old compiled assets
rake assets:clobber             # Remove compiled assets
rake assets:environment         # Load asset compile enviro..
rake assets:precompile          # Compile all the assets ...
rake cache_digests:dependencies # Lookup first-level depen...
rake db:_dump                   #
rake db:charset                 #
...
```
If you want to make a custom Rake task in your Rails application you can create a file with the `.rake`extension in the `lib/tasks` folder. For instance if you would like to make that import inside a Rails application,  you would add the code from yesterday to `lib/tasks/import.rake`.
One important part we haven't talked about yet is namespacing. There might be another task callled `download` in your Rails application which would cause some unexpected results. Important to note here the fact, if Rake finds a task definition for an already defined task it will extend not override it. To make sure our tasks are safe and also to make them nicely grouped we will move them into a namespace:

```ruby
# lib/tasks/import.rake
namespace :import do
  desc "Download data"
  task :download do
    puts "download data from source"
  end

  desc "Import categories"
  task :categories => :download do
    puts "import categories"
  end

  desc "Import products"
  task :products => :categories do
    puts "import products"
  end

  desc "Import orders"
  task :orders => :download do
    puts "import orders"
  end

  desc "Import eveything"
  task :all => [:categories, :products, :orders] do
    puts "Imported everything"
  end
end
```
From now on we can call the tasks with the colon syntax: `rake import:all`.  When you write a task which interacts with Rails you should set the `environment` task to a dependency. Since the `download` task is called in each case we can make that depending on `environment`:

```ruby
...
task :download => :environment do
  puts "download data from source"
end
...
```
This task basically loads your app for you in the environment you had set.
It is good to know Rails sets `eager_load` to false in this task so if you want to use multiple threads in a task it is better to not use this task as a dependency, rather just load the environment yourself:

```ruby
...
# if you use multiple threads
task :download do
  Rails.application.require_environment!
  puts "download data from source"
end
...
```

This was a short one but tomorrow we will do some really interesting stuff when I will share you how I build this newsletter with the help of Rake.
Here is the full Rakefile of what we have done today:

```ruby
# lib/tasks/import.rake
namespace :import do
  desc "Download data"
  task :download => :environment do
    puts "download data from source"
  end

  desc "Import categories"
  task :categories => :download do
    puts "import categories"
  end

  desc "Import products"
  task :products => :categories do
    puts "import products"
  end

  desc "Import orders"
  task :orders => :download do
    puts "import orders"
  end

  desc "Import eveything"
  task :all => [:categories, :products, :orders] do
    puts "Imported everything"
  end
end
```
