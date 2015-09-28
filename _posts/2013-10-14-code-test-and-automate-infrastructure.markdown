---
layout: post
title: "code test and automate infrastructure"
date: 2013-10-14 16:36
comments: true
categories: 
permalink: /blog/2013/11/07/code-test-and-automate-infrastructure/
---


Infrastructure as code is mantra these days and is often japas by using various frameworks like chef puppet and ansible etc...
this area {" cross cuts between developers and ops domains"}, and still the patterns for infrastructure code are still evolving.

This post is about using chef and apply best practices for code test and automate our and our clients infrastructure.

<!--more-->

## Part One - The Code

Code should be extensible, maintainable , reusable and in short modular but keeping all chef code in single repository have some issues and it
also make code reuse difficult.

##Problems:

* code and data is in same place , but they often evolves with different pace
* cookbooks in one big repository are difficult to develop and test independently
* cookbooks in same repository are not modular and difficult to
  distribute and reuse.


##Solution

To solve these problem there are some frameworks, we use berkself.
it allows to use and distributes cookbooks similar to gems.
it is not only nicely written but is a foundation of software principals, There are numerous articles on this and we are also using berkself


*We take care of following points while developing chef code.*

* use community cookbooks and write wrapper cookbook for extension if
  required
* use no role, roles are like global variables, also if required write aggregated recipes
* use convention over configuration with proper naming convention , use search and sane defaults
* create application cookbook with git repo for itself for each project.

following are the patterns , which i regularly encounter while writing chef code.

## Patterns

* Aggregated Recipe 

These are equivalent to aggregation, and only includes other recipes.
for example default recipes can contain app and db recipes and then can
be used for setting dev testing box.

{% highlight ruby %}
#application cookbook: default.rb
include_recipe 'app'
include_recipe 'db'
{% endhighlight %}

* Wrapper Recipe

 this is like decorator and can be used for extending community cookbook.

{% highlight ruby %}
 # foo.rb
include_recipe 'foo'

bar "configuration for foo" do
end
{% endhighlight %}


* Private Recipe

 this is like abstract class and contains code shared by recipes which are exposed to public,
 generally it's name is prefixed with _ , it can also replaced by extracting common
 code in lwrp or definitions.

* Application cook book

this is actual cookbook for project , I prefer to create one cook for
each micro rest component and different recipes in it for infrastructure needs. 

this contain application specific configuration code and often use
abstracted code from library cookbook's lwrps and definition.

recipes are created for each component like db , app , cache , proxy , elb so on
{% highlight shell %}
#application cookbook dir structure
.
|-- Berksfile
|-- Vagrantfile
|-- attributes
|   |-- app.rb
|   `-- db.rb
|-- metadata.rb
`-- recipes
    |-- app.rb
    |-- db.rb
    `-- redis.rb
{% endhighlight %}
though micro service tempt to reuse , one project ,using tag , what this often leads to conditional code in end

* Library Cookbook

this is brain and heart of chef code base , contain all common code for
application cookbooks.
library cookbook contain no recipe, it abstract common code for reusing in application cook books in form of lwrps and definitions
for example set up app servers (puma/nginx combo , torquebox ), app rpm/omnibus installers

it can be combined with base cookbook also.

* Base Cookbook

It replaces the base role and install basic packages like vim , tmux/screen , yum repos , environment specific tools like debug, tracing tools, monitoring clients etc . the benefit over role is that you can version it and also different recipes for environment provides more flexibility.

{% highlight shell %}
.
|-- Berksfile
|-- Vagrantfile
|-- attributes
|   |-- stagging.rb
|   `-- production.rb
|-- metadata.rb
`-- recipes
    |-- _common.rb
    |-- stagging.rb
    `-- production.rb
{% endhighlight %}

* Infrastructure Cookbook

this contain wrapper recipes/scripts for common infrastructure tools chef server, bind , openvpn , ganglia server and ci and often use wrapper recipes over community cookbooks. 

this provide two benefits
1) one place for all common infrastructure code.
2) own name space which is useful for searching and setting fqdn etc in
application cookbooks.



Part two is about testing cookbook and setting up CI and build
pipelines.
