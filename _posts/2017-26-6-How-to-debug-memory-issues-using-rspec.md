---
layout: post
title: How to debug memory issues using RSpec?
---

## problem

Long story short: **CI** server build started to fail all the time due to **memory** consumption. This is why I started dig into the problem try to find out the reason of the memory leak - is it my **tests** or my **code**? Let's find out!

The most user friendly gem I found is [get_process_mem](https://github.com/schneems/get_process_mem) gem which can tell you how many **memory** a process is using. After you do add `gem 'get_process_mem'` to your `Gemfile` and `bundle install` you can check how much **memory** your code uses:

```ruby
[2] pry(main)> GetProcessMem.new.mb
=> 140.96484375
[3] pry(main)> 100.times { Object.new }
=> 100
[4] pry(main)> GetProcessMem.new.mb
=> 141.671875
```

I wanted to track where do I have memory leak so I added snippet to the `spec/rails_helper.rb` which will log how much memory is used after each spec execution.

```ruby
  RSpec.configure do |config|
  ...
  require 'csv'

  config.around(:each) do |example|
    example.run
    CSV.open("memory.csv", "a") do |csv|
      csv << [example.location, GetProcessMem.new.mb]

    end
  end
  ...
  end
```

Then you just execute in terminal:

```
$ rspec spec
```

I can build graph with generated file to find the spikes:


I search a lot in both google and here in SO, but I did not find the answer.

I use jekyll, and in the about-me page, I what to embed this , the given iframe looks like this:

{% raw %}
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" src="https://plot.ly/~denys.medynskyi/2.embed"></iframe>
{% endraw %}



