---
layout: post
title: "code test and automate infrastructure - part II"
date: 2013-10-15 18:54
comments: true
categories: 
permalink: /blog/2013/12/09/code-test-and-automate-infrastructure-part-ii/
---


## Part Two  - Testing

Code evolve with tests and CI is vital part of it if multiple developers working in same code base,

As in first part I mentioned to use one repo for each project, we can use execute tests in ci for every commit and
generate artifacts which are cookbook and environment.

Recent version of chef spec is really good for unit tests and test
kitchen is for running integration tests in isolated environment.

Community ntp cookbook is great starting point for writing tested cookbooks


pipeline for an application cookbook is as follow

in short testing part is covered by:

* unit tests with chef spec
* rubocop , foodcritics for lint
* integartion tests with test kitchen and bats

<!--more-->

so each commit in application cookbook trigger bulid pipeline and
execute stages in order.

we have inregarated docker in ci serevr and using  test kitchen with docker plugin for running integtarion tests.

vagrant lxc , intgratio tests
test kitchen , spec
chef zero

using application cook book for each project

final outcome:
 artifacts
 cookbook , environment




