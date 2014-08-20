# Hello

Just a quick summary of what will be covered in this course:

Day 1: Introduction to Rake

Day 2: Dependencies

Day 3: Rails and Rake

Day 4: Use Rake to build a project

## What is rake?
Rake is a Rubygem written by Jim Weirich. He built the ruby version of `make` (the C build tool) and over time Rake became a popular tool to create command line Ruby applications and automation scripts. It has an easy to follow DSL and some very powerful features like task dependencies, but more on these later.

## What can I use Rake for?
If you know Ruby on Rails you use Rake very often because most of the command line Rails features are Rake tasks. Namely `rake db:migrate`, `rake test` etc. If you need a custom command line script for your Rails application (running an import, periodically updating something in the database) Rake is your friend.
Also anytime you want to automate something with a script, Rake is a very powerful tool to rely on.

## Install rake

Rake is distributed as a Rubygem so all you need to do is enter `gem install rake` in your console to install it.

## Rakefile
Rake reads the task from a Rakefile and in additionally you can separate your tasks into multiple files and put them into the `rakelib` folder with the `.rake` extension.
Let’s create a Rakefile with a simple task:

```ruby
# Rakefile
desc "Hello world"
task :hello_world do
  puts "Hello world!"
end
```
Now we know how to make 'Hello world' with Rake. Let's break down this code. First we give a description to the task than we set the name of the task to a symbol(we could you strings too) than passing a block and Rake will evaluate that block when we call this task.
![image1](../images/image1.jpg)

Let's call the task:

```shell
$ rake hello_world
Hello world!
```
If you enter `rake -T` in your console you will get a list of available tasks:

```shell
$ rake -T
rake hello_world  # Hello world
```
The Rakefile is basically a ruby file so you can write any ruby code inside.
## Task types
Rake has four types of tasks:

 * `task` is a generic task
 * `directory` creates a directory
 * `file` creates a file with the result of the block we pass
 * `multitask` calls its dependencies in parallel

That's it for today, tomorrow you will learn about dependencies.
