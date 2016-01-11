---
layout: post
title:  "Adding 3-dimensional search to your Rails application"
date:   2016-01-12 11:00:00
categories: "ruby on rails"
---

One time when I was working on a research project, I had to find the materials within a L\*a\*b\* color space distance. This applies to anyone who needs to find a quick way to filter through objects through a 3-dimensional distance vector. You may want to do the following but it is not recommended:

{% highlight ruby %}
def self.filter_by_color params
  where(' ((color2_l - ?) ^ 2 + (color2_a - ?) ^ 2  + (color2_b - ?) ^ 2) < ?', params[:L], params[:a], params[:b], params[:max_distance].to_f**2 )
end
{% endhighlight %}

What's wrong with this approach is that you are using the application to process all the colors of each object instead of the database. Using Ruby to do this takes an excruciating long amount of time to process. 
You should delegate this job to the database with all its indices set.

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

And you are done! 

In this context, the 3d vector consist of L,a and b values. We find the materials that are within the radius of ```params[:max_distance]``` ordered by their distance from the current object that is running this method.

Hopefully this helps anyone who is in need of a solution to a similar problem.