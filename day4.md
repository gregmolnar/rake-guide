# Taking it to the next level


As I already mentioned Rake is intended to be a build tool, but it is used to do a lot of other things and many Rake users don't even know the features makes Rake a powerful build system.
Actually I am using Rake to build this course. I write all the content in markdown, and I have a Rake task which converts that to HTML.
I have a directory structure like this:

```
source
 - day1.md
 - day2.md
 - day3.md
 - day4.md
 ...
Rakefile
```
As you see, I store my markdown files in the source folder, and I want to have an `output` folder where I want my HTML files to be created. Rake has a directory method which I can use:

```ruby
directory "output"
```
Next, I will create a Filetask, which is a bit different from normal tasks. Filetasks doesn't trigger all the time, only if the target file doesn't exist or the source file is changed since the target was created. Because of this Rake will save some time on rebuilds. Let's see some code for this:

```ruby
# Rakefile
directory "output"

desc "build day1"
file "output/day1.html" => ["output", "source/day1.md"] do |task|
  rendered = redcarpet.render(
    File.read(task.prerequisites.first)
  )
  File.new(task.name, 'w+').write(rendered)
  puts "Built #{task.name}"
end
```
When you define a Filetask, the name of the task is the target file and you can pass an array of prerequisites with the hashrocket and do the work in the block. The prerequisites are the `output` folder and the source file in this case.
With the file task I have access to the [Task](http://rake.rubyforge.org/Rake/Task.html) object in the block which holds the name of the task and the prerequisites. I print the name of the task at the end of the block which helps to see when the file is actually built. If I call `rake output/day1.html` twice in a row, the second time it won't print the output because it will not generate the file again. Two more things to note about Filetasks:

* the naming convention is to use strings but you can use symbols if you would like
* Filetasks ignore the namespaces

If I would go this way I should repeat this code for each day  and I am too lazy to do that. Instead I use Rake's FileList and I create an array of arrays with the target and the source:

```ruby
# Rakefile
directory "output"

source_files = FileList['source/*.md']
target_files = source_files.pathmap("output/%n.html")

target_files.zip(source_files).each do |target, source|
  desc "build #{target}"
  file target => ["output", source] do |task|
    rendered = redcarpet.render(
      File.read(task.prerequisites.first)
    )
    File.new(task.name, 'w+').write(rendered)
    puts "Built #{task.name}"
  end
end
```
If I list the tasks now I get the following result:

```shell
$ rake -T
rake output/day1.html  # build output/day1.html
rake output/day2.html  # build output/day2.html
rake output/day3.html  # build output/day3.html
rake output/day4.html  # build output/day4.html
```
Since I don't want to trigger these individually, I have a task to build all of the source files:

```ruby
desc "Build all files"
task :build => target_files
```
As you I see I just simply set `target_files` as a dependency and Rake will call all the dynamically generated tasks for me.
This isn't a bad solution but if I had hundreds of files it wouldn't be optimal. Instead Rake has another way of achieving the same result. It is called Rules.

### Rules
This feature of Rake is a little bit like `method_missing` in Ruby. If Rake couldn't find a Filetask it will look at the rules defined and if it finds a match, it will call it. When we define a rule we pass the target as either a string or a regular expression, and we also pass the source, but since that is dynamic we will use a lambda to get the source name from the target. Then in the block we will do the work:

```ruby
# Rakefile
directory "output"
source = lambda { |fn|
 fn.pathmap("source/%n.md")
}
rule /^output\/.*\.html/ => [source] do |task|
  rendered = redcarpet.render(File.read(task.source))
  File.new(task.name, 'w+').write(rendered)
  puts "Built #{task.name}"
end

task :build => ["output"] + target_files
```

Now we have a few lines of code which covers almost all my needs. One more thing I'd like to have is a way to get a clean state, so I will use Rake's built in clean task:

```ruby
require 'rake/clean'
CLEAN.include("output")
```

# Summary

That's all I wanted to share with you about Rake. I hope you enjoyed the series of emails and you learned a few tricks.
If you have any questions of feedback about what you've read don't hesitate to contact me at [greg@molnar.io](mailto:greg@molnar.io).
