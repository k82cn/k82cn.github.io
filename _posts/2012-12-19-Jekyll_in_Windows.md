---
layout: post
categories: tech
title: Jekyll in Windows
---

Jekyll is a simple, blog aware, static site generator. It takes a template directory (representing the raw form of a website), runs it through Markdown or Textile and Liquid converters, and produces a static website suitable for any web server.

## Downloads

* [Ruby 1.9.2](http://cdn.rubyinstaller.org/archives/1.9.3-p327/rubyinstaller-1.9.3-p327.exe)
* [PortablePython 3.2.1.1](http://elvis.rowan.edu/mirrors/portablepython/v3.2/PortablePython_3.2.1.1.exe)
* [DevKit](https://github.com/downloads/oneclick/rubyinstaller/DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe)

## Install DevKit

    ruby dk.rb init
    ruby dk.rb install

## Install jekyll

    gem install jekyll

## Install rdiscount

    gem install rdiscount

## Set System code

    set LC_ALL=en_US.UTF-8
    set LANG=en_US.UTF-8

## Start local Jekyll server

    jekyll --server
