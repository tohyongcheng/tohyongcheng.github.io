---
layout: post
title:  "Adding 3-dimensional search to your Rails application with PostgreSQL"
date:   2016-01-12 11:00:00
categories: "ruby on rails"
---

One time when I was working on a research project, I had to find the materials within a L\*a\*b\* color space distance. This applies to anyone who needs to find a quick way to filter through objects through a 3-dimensional distance vector. You may want to do the following but it is not recommended:

{% highlight ruby %}
def self.filter_by_color params
  where(' ((color2_l - ?) ^ 2 + (color2_a - ?) ^ 2  + (color2_b - ?) ^ 2) < ?', params[:L], params[:a], params[:b], params[:max_distance].to_f**2 )
end
{% endhighlight %}

There is actually nothing wrong with this approach. What we are doing here, is that we are already using an SQL query to help us determine what we need to find. However, it just doesn't look semantically nice, and that there should be a better way to process this. Furthermore, I had to be able to order by the distance, which in this case I was unable to. 

What I did was simply to activate an extension for PostgreSQL. First, generate a migration file.

```
rails generate migration AddCubeExtension
``` 

Then add the following code to the migration file.

{% highlight ruby %}
class AddCubeExtension < ActiveRecord::Migration
  def change
    enable_extension 'cube'
  end
end
{% endhighlight %}

More details on the functions provided by the extension can be found [here](http://www.postgresql.org/docs/9.4/static/cube.html).

You will then need to add an array field of decimals/integers to the model that stores your 3d vector:

{% highlight ruby %}
# rails generate Add3DVectorToMaterial

class Add3DVectorToMaterial < ActiveRecord::Migration
  def change
    change_column :materials, :color_vector, :decimal, array: true, default: []
    add_index  :materials, :color_vector, using: 'gin'
  end
end

{% endhighlight %}

Migrate the database:

```
rake db:migrate
```

Followed by adding the following code to the model:

{% highlight ruby %}
class Material < ActiveRecord::Base

  [...]

  def self.filter_by_color params
    color_vector_string = [params[:L], params[:a], params[:b]].map(&:to_s).join(',')
    self.select("*,
      cube_distance(
        cube(color_vector),
        '(#{color_vector_string})'
      )
      AS \"color_distance\"")
    .where("cube_distance(cube(color_vector),'(#{color_vector_string})') < ?", params[:max_distance])
    .order("color_distance")
  end

end
{% endhighlight %}

And now I can also order by color_distance!

In this context, the 3d vector consist of L,a and b values. We find the materials that are within the radius of ```params[:max_distance]``` ordered by their distance from the current object that is running this method.

Using a benchmark, it seems that using cube_distance helped to increase the speed by a little:

{% highlight ruby %}
require 'benchmark/ips'
Benchmark.ips do |x|
   x.config(time: 5, warmup: 2)
   params = {L: 100, a: 100, b: 100, max_distance: 100}
   x.report("using normal SQL") { Material.filter_by_color_old params }
   x.report("using cube") { Material.filter_by_color params }
   x.compare!
end
{% endhighlight %}

```
Calculating -------------------------------------
    using normal SQL     1.151k i/100ms
          using cube     1.130k i/100ms
-------------------------------------------------
    using normal SQL     28.481k (± 9.3%) i/s -    141.573k
          using cube     28.692k (±10.1%) i/s -    142.380k

Comparison:
          using cube:    28692.1 i/s
    using normal SQL:    28480.6 i/s - 1.01x slower

 => #<Benchmark::IPS::Report:0x007fc84b8f5ac8 @entries=[#<Benchmark::IPS::Report::Entry:0x007fc845bf7128 @label="using normal SQL", @microseconds=5016804.6951293945, @iterations=141573, @ips=28480.570212955136, @ips_sd=2659, @measurement_cycle=1151, @show_total_time=false>, #<Benchmark::IPS::Report::Entry:0x007fc843f5e020 @label="using cube", @microseconds=5016709.804534912, @iterations=142380, @ips=28692.12437310675, @ips_sd=2904, @measurement_cycle=1130, @show_total_time=false>], @data=nil> 
```

It seems that it doesn't really affect the performance much. But oh well, at least I can order it by distance from a point now. Hopefully this helps anyone who is in need of a solution to a similar problem.