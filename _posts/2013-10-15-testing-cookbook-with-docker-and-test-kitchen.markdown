---
layout: post
title: "Testing chef cookbook with docker and test kitchen"
date: 2013-10-15 17:44
comments: true
categories: [devops,tdd]
permalink: /blog/2013/10/15/testing-cookbook-with-docker-and-test-kitchen/
---

[Test Kitchen](https://github.com/opscode/test-kitchen) is a framework for isolated integration testing of chef
recipes.
 
For testing a recipe it spawn a vm, execute tests and then destroys it.
For local cookbook development and manual testing of changes
[vagarnt](http://www.vagrantup.com/) is definitely the first choice,
but for developing cookbook with automated tests suits this VM based approach is very
slow and as test kitchen destroys box with each run, testing feedback time become really important factor in development speed and is worthy candidate for
optimisation.


## Optimisation
Containers like openvz and lxc are faster to launch and are very lightweight as compared to virtual box and other VM based backends. 

As compared to open vz, lxc is available on the mainstream linux kernel but managing lxc container with scripts
is not an easy task, there are two lxc baced framework available, [vagrant lxc](https://github.com/fgrehm/vagrant-lxc)(vagrant lxc provider) and [docker](https://www.docker.io/) (package and run application as container) which provides 
good abstraction layer over lxc.

Test kitchen has a architecture for pluggable virualization backend and it support vagrant, ec2 and recently
with [kitchen-docker](https://github.com/portertech/kitchen-docker) plugin, docker can also used as driver.

<!--more-->

I am using following setup to use test kitchen with docker.

##Setup

###Install docker

[Install](https://www.docker.io/gettingstarted/) it for supported
platform or
Install it with community cookbook with chef and berkself

{% highlight ruby %}
#Berksfile to install docker
site :opscode
metadata
group :integration do
cookbook 'docker'
end
{% endhighlight %}
 
### Install test-kitchen and docker driver

Can be installed with bundler by using following Gemfile file

{% highlight ruby %}
source 'https://rubygems.org'

gem 'berkshelf', '~> 2.0'

group :integration do
  gem 'test-kitchen', '~> 1.0.0.beta'
  gem 'kitchen-docker'
end
{% endhighlight %}

### Test Cookbook 

Download [ntp](https://github.com/opscode-cookbooks/ntp.git) cookbook for testing, beacuse it also serve as a testing documentation reference

##Execute Tests

Change .kitchen.yml file of ntp cookbook to use kitchen docker plugin

{% highlight yaml %}
---
driver_plugin: docker
driver_config:
  require_chef_omnibus: true

platforms:
- name: centos
  driver_config:
    image: "centos"
    platform: "rhel"
  run_list:
  - recipe[yum]

suites:
  - name: default
    run_list:
      - recipe[ntp::default]
    attributes:
      ntp:
        sync_clock: true
        sync_hw_clock: true
  - name: undo
    run_list:
      - recipe[ntp::undo]
{% endhighlight %}

And run kitchen to execute tests in docker container

{% highlight yaml %}
bundle exec kitchen test
{% endhighlight %}


## More optimisations

Test Kitchen downloads chef omnibus package every time while spawning
a container, this step takes both time and bandwidth, this behaviour can
be override by setting require_chef_omnibus flag, there are also few tricks to speed up this step.

* use local repository for chef omnibus and override chef_omnibus_url flag
* use lightweight gem like chef zero
* utilize docker cache with  provision_command command
* package omnibus with container and build image for testing

New image can be easily created by using following
docker file 

{% highlight shell %}
cat << 'EOF' > Dockerfile
FROM centos
RUN curl -L https://www.opscode.com/chef/install.sh | sudo bash
EOF
docker build -t  image_name .
{% endhighlight %}

Set and image: image_name in .kitchen.yml
file and execute tests more faster.
