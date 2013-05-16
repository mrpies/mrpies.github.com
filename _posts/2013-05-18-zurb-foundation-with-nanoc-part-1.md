---
layout: default
title: Using Zurb Foundation 3 and Nanoc - Part 1
abstract: Installation and Environment Setup
published: true
---

> Note: These instructions are from my testing on Mac OS X - they might need some tweaking for Windows/Linux.

### Installing nanoc and Creating a Project

Following the instructions in the [nanoc tutorial](http://nanoc.ws/docs/tutorial/), I'm creating a new site:

{% highlight bash %}

$ gem install nanoc adsf
$ nanoc create-site vintageinvite
...
$ cd vintageinvite
vintageinvite$ ls
Rules   content   layouts   lib   nanoc.yaml  output
vintageinvite$ nanoc compile
...
vintageinvite$ nanoc view
{% endhighlight %}

This shows the default nanoc new project landing page at `localhost:3000/`. Sweet.

### Setting Up Dependencies

The next job is to set up the environment, starting by setting up [bundler](http://gembundler.com) for the ruby libraries I'll be depending on, using a Gemfile (edit using your favourite editor, subl here refers to [Sublime Text 2](http://www.sublimetext.com)):

{% highlight bash %}
vintageinvite$ gem install bundler    
vintageinvite$ subl Gemfile
{% endhighlight %}

And inside `Gemfile`:

{% highlight ruby %}
source :rubygems

gem 'nanoc'
gem 'adsf'
gem 'haml'
gem 'zurb-foundation', '~> 3.2.5'
gem 'compass'
{% endhighlight %}

{% highlight bash %}
vintageinvite$ bundle install
Resolving dependencies...
Using rake (10.0.4) 
Using rack (1.5.1) 
Using adsf (1.1.1) 
Using chunky_png (1.2.8) 
Using colored (1.2) 
Using fssm (0.2.10) 
Using sass (3.2.7) 
Using compass (0.12.2) 
Using cri (2.3.0) 
Using tilt (1.3.6) 
Using haml (4.0.2) 
Using sassy-math (1.5) 
Using modular-scale (1.0.6) 
Using nanoc (3.6.2) 
Using zurb-foundation (3.2.5) 
Using bundler (1.3.0) 
Your bundle is complete! Use \`bundle show \[gemname\]\` to see where a bundled gem is installed.
{% endhighlight %}
<span class="code-link"><a href="https://github.com/mrpies/vintageinvite/blob/v1.0/Gemfile">Gemfile</a></span>

The gems we're using are: 

  * _nanoc and [adsf](http://stoneship.org/software/adsf/)_ - adsf (we installed above) is **a** **d**ead **s**imple **f**ileserver used for `nanoc view`
  * _[haml](http://haml.info/)_ - is my preferred html abstraction, which we'll talk about later
  * _zurb-foundation_ - contains everything you need to use Foundation
  *  _[compass](http://compass-style.org/)_ - is used by Foundation, but also will help us in other ways (more later)

### Assets: css, js and images

As a Rails user, I quite enjoy how the Rails asset pipeline works, so I'll be mirroring that directory structure in this project. This needs a few new directories, and removal of the default `stylesheet.css` provided with nanoc:

{% highlight bash %}
vintageinvite$ mkdir -p content/assets/stylesheets
vintageinvite$ mkdir -p content/assets/javascripts
vintageinvite$ mkdir -p content/assets/images
vintageinvite$ rm content/stylesheet.css
{% endhighlight %}

### Compass and Foundation Integration

To [integrate compass](http://compass-style.org/help/tutorials/integration/) with nanoc, I need to create a `compass.rb` file:

{% highlight bash %}
vintageinvite$ subl [compass.rb](https://github.com/mrpies/vintageinvite/blob/v1.0/compass.rb)
{% endhighlight %}

{% highlight ruby %}
require "zurb-foundation"

# -----------------------------------------------
# Paths
# -----------------------------------------------

http_path = "/"
css_dir = "output/assets"
images_dir = "content/assets/images"
javascripts_dir = "content/assets/javascripts"
sass_dir = "content/assets/stylesheets"

# -----------------------------------------------
# Output
# -----------------------------------------------

output_style = :compressed
preferred_syntax = :scss
relative_assets = true
{% endhighlight %}


We're telling compass where the necessary directories are for content, as well as the output directory for generated stylesheets (though we aren't using all of the power of compass, we need to make sure these directories are correct for the next step).

Now we can use compass to install Foundation in our project:

{% highlight bash %}
vintageinvite$ bundle exec compass install foundation -c compass.rb
{% endhighlight %}

This adds some unnecessary files that we can remove (we'll be adding robots.txt in the correct area later)

{% highlight bash %}
vintageinvite$ rm index.html humans.txt robots.txt MIT-LICENSE.txt 
{% endhighlight %}

The final configuration step for is changing the nanoc `Rules` file, to include compass and to process our assets directory correctly:

[Rules](https://github.com/mrpies/vintageinvite/blob/v1.0/Rules)
{% highlight bash %}
vintageinvite$ subl Rules
{% endhighlight %}

{% highlight ruby %}
require 'compass'

Compass.add_project_configuration 'compass.rb'

compile '/assets/stylesheets/*' do
  filter :sass, Compass.sass_engine_options.merge(:syntax => item[:extension].to_sym)
end

route '/assets/images/*' do
  #ignore images for now
end

route '/assets/javascripts/*' do
  #ignore javascripts for now
end

route '/assets/stylesheets/*' do
  # flatten assets to assets/ similar to rails
  '/assets/' + File.basename(item.identifier) + '.css'
end
{% endhighlight %}

We've added a step to filter the sass stylesheets in `assets/stylesheets`, as well as a new route to push all assets into the root of the `output/assets` directory (mainly because this is how Rails does it, and I like it).

Now we can change our default layout to include the correct stylesheet, and use Foundation styles:

{% highlight bash %}
$ subl layouts/default.html
{% endhighlight %}

{% highlight rhtml %}
<!DOCTYPE HTML>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>A Brand New nanoc Site - <%= @item[:title] %></title>
    <link rel="stylesheet" href="/assets/app.css">
  </head>
  <body>
    <div class="row">
      <div class="twelve columns">
        <%= yield %>
      </div>
    </div>
  </body>
</html>
{% endhighlight %}

{% highlight bash %}
$ nanoc compile
$ nanoc view
{% endhighlight %}

Which should produce this:

![Foundation Enabled](/asset/image/2013-05-18/zurb-nanoc-1.png "Foundation laid...")

### Haml Layouts

Having used it for a while in Rails, I can firmly say I'm a big fan of haml. It reduces the verbosity of markup to the extent that it almost looks nice!

To switch the layouts from erb enabled html we just need to change the layout line in `Rules` to the following:

{% highlight ruby %}
layout '*', :haml, :format => :html5
{% endhighlight %}

And now we need to change the default layout - I've also added a simple Foundation top-bar nav:

{% highlight bash %}
vintaginvite$ mv layouts/default.html layouts/default.haml
vintaginvite$ subl layouts/default.haml
{% endhighlight %}

[layouts/default.haml](https://github.com/mrpies/vintageinvite/blob/v1.0/layouts/default.haml)

{% highlight haml %}
!!! 5
%html
  %head
    %meta{:charset => "utf-8"}
    %title
      A Brand new nanoc site -
      = item[:title]
    %link{:rel => "stylesheet", :href => "/assets/app.css" }
  %body
    .header
      .contain-to-grid
        %nav.top-bar
          %ul
            %li.name
              %a{:href => "/"} the vintage invite co
            %li.toggle-topbar
              %a{:href => "#"}
          %section
            %ul.right
              %li
                %a{:href => "#"} Contact
    .row
      .twelve.columns
        = yield
{% endhighlight %}

{% highlight bash %}
$ nanoc compile
$ nanoc view
{% endhighlight %}

And that gives us this:

![Haml'ed](/asset/image/2013-05-18/zurb-nanoc-2.png "Ready for the next steps!")