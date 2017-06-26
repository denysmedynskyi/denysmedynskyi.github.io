---
layout: post
title: How to debug memory issues using RSpec?
---

[Long story short](https://www.youtube.com/watch?v=Lbz2CZXXLMM): I had a **memory** issue.

## problem

All **CI** builds were failing with an exception `Cannot allocate memory`, which means that server does not have enough **memory**. Of course, you can just increase **memory** on the server, but it can be a sign of the problem, so it is worth to be investigated. This is why I started digging into the problem to find out the reason of the **memory leak**.

## research

The most user friendly gem I found is [get_process_mem](https://github.com/schneems/get_process_mem) which can tell you how much **memory** a process uses. Add `gem 'get_process_mem'` to your `Gemfile` and after `bundle install` you can try it:

```ruby
$ rails c
[1] pry(main)> GetProcessMem.new.mb
=> 140.96484375
[2] pry(main)> 100.times { Object.new }
=> 100
[3] pry(main)> GetProcessMem.new.mb
=> 141.671875
```

I needed to inspect RSpec test suite, so I added the following snippet to the `spec/rails_helper.rb`:

```ruby
  RSpec.configure do |config|
    ...
    require 'csv'

    config.around(:each) do |example|
      # run example
      example.run
      CSV.open("memory.csv", "a") do |csv|
        # log memory to the CSV file or you can write to text file
        csv << [example.location, GetProcessMem.new.mb]
      end
    end
    ...
  end
  ```

Then execute in the terminal:

`$ rspec spec`

Result is a CSV file with values: example & memory consumption in MBs:

```
...
./spec/requests/public/insights_home_page_request_spec.rb:50  158.7
./spec/requests/public/insights_home_page_request_spec.rb:5  157.1
./spec/requests/public/insights_home_page_request_spec.rb:14  188.76
...
```

## investigation

To solve this puzzle you need to look for the spikes - large increases in memory. You can do it manually of using [plot tool](https://plot.ly/~denys.medynskyi/2.embed):

![Request specs graph](http://i.imgur.com/2NmgIuv.png)

Here is my spike:

![spike](http://i.imgur.com/IfEciet.png)

After you find spikes try to find similarities between them.

Do you have a suspect? Check if it leaks:


```ruby
$ rails c
1] pry(main)> Rails.logger.level = :warn # to remove distractions
=> :warn
[2] pry(main)> GetProcessMem.new.mb
=> 157.37109375
[3] pry(main)> 10.times do |iteration|
[3] pry(main)*   Pages::Seed.create_generic_page(:home)
[3] pry(main)*   p "##{iteration}=#{GetProcessMem.new.mb}"
[3] pry(main)* end
"#0=181.0625"
"#1=200.68359375"
"#2=219.23828125"
"#3=221.09765625"
"#4=222.1875"
"#5=222.9921875"
"#6=223.94140625"
"#7=224.8046875"
"#8=225.21484375"
"#9=225.296875"
=> 10
```

As you can see memory stops growing, so it is a [memory bloat](http://book.scoutapp.com/memory-bloat.html).

## conclusion

After the reason for memory growth is found next step is to decide if is it worth to fix it or not.

**In my case**: 
1. I suspected problem in the specs, but found that memory increases the same way in `rails console`
2. It is not a memory leak
3. Memory bloat is small enough to fix it later  

Thank you for reading! I hope it was useful for you.
