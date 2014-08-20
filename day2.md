# Dependencies

Today I will teach you how Rake handles the dependencies. I will use code examples but instead of repeating the whole code everytime I will just show you the important parts, then at the bottom of the email you can view the full example.
Rake makes it easy to define dependent tasks. Let's say we want to import a dataset from a remote source into our application. We want to make separate tasks for importing the categories, products and the orders so we can run those bits separately:

```ruby
desc "Import categories"
task :categories do
  puts "import categories"
end

desc "Import products"
task :products do
  puts "import products"
end

desc "Import orders"
task :orders do
  puts "import orders"
end
```
So far so good but we need to get the remote data before each of these tasks so we will define another task and set that as a dependency for these:

```ruby
desc "Download data"
task :download do
  puts "download data from source"
end

desc "Import categories"
task :categories => :download do
  puts "import categories"
end

desc "Import products"
task :products => :download do
  puts "import products"
end

desc "Import orders"
task :orders => :download do
  puts "import orders"
end
```
![dependency](../images/image2.jpg)
As you see you can set a dependency with the hashrocket and you can pass one task or an array of tasks. If you call rake with the -P switch it will display the dependency tree:

```shell
$ rake -P
rake categories
    download
rake download
rake orders
    download
rake products
    download
```
In our example, when we import the products, we want to make sure we have the latest categories so let's make the products task depend on categories:

```ruby
desc "Download data"
task :download do
  puts "download data from source"
end

desc "Import categories"
task :categories => :download do
  puts "Import categories"
end

desc "Import products"
task :products => :categories do
  puts "Import products"
end

desc "Import orders"
task :orders => :download do
  puts "import orders"
end
```
This looks great but we would like to have a task to run a full import too so let's make a task for that:

```ruby
desc "Import everything"
task :all => [:categories, :products, :orders] do
  puts "Imported everything"
end
```
![dependency](../images/image3.jpg)

You might think: "But wait... this will run `download` 3 times and `categories` 2 times, won't it?" Nope. Rake disables a task after the first invocation. If you want to run a dependent task even if it is already invoked you can `reenable` it.
If you call the task you should see this result:

```shell
$ rake all
download data from source
import categories
import products
import orders
Imported everything
```
If you want to import the products only:

```ruby
$ rake products
download data from source
import categories
import products
```
If you need to `reenable` a task:

```ruby
desc "Import categories"
task :categories => :download do
  puts "Import categories"
  Rake::Task["categories"].reenable
end
```
Note that it imported the categories first because products depends on categories.
We can also set a default task with rake so when you call only `rake` it will run the default task. Let's set the `all` task to be the default:

```ruby
task :default => [:all]
```
As I promised here is how the Rakefile should look now:

```ruby
desc "Download data"
task :download do
  puts "download data from source"
end

desc "Import categories"
task :categories => :download do
  puts "Import categories"

end

desc "Import products"
task :products => :categories do
  puts "Import products"
end

desc "Import orders"
task :orders => :download do
  puts "import orders"
end

desc "Import eveything"
task :all => [:categories, :products, :orders] do
  puts "Imported everything"
end

task :default => [:all]
```
That's it for today. Tomorrow we will look at how Rails uses Rake and how can we add custom Rake tasks to a Rails application.
