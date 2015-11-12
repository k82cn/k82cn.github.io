---
layout: post
categories: blogs
title: Selenium Python in console
---

Selenium a powerful suite of tools for web testing, but it’s dependent on browser (Firefox, IE, Chrome); and those browser need a displayer. As a console/command geek, it’s intolerable. After several days investigation, I’d like to introduce [PyVirtualDisplay](http://pypi.python.org/pypi/PyVirtualDisplay) to run Selenium in a console with Python.

## Selenium introduction

[Selenium](http://seleniumhq.org/) automates browsers. That’s it. What you do with that power is entirely up to you. Primarily it is for automating web applications for testing purposes, but is certainly not limited to just that. Boring web-based administration tasks can (and should!) also be automated as well.

Selenium has the support of some of the largest browser vendors who have taken (or are taking) steps to make Selenium a native part of their browser. It is also the core technology in countless other browser automation tools, APIs and frameworks.

## Selenium in console

Selenium a powerful suite of tools for web testing, but it’s dependent on browser (Firefox, IE, Chrome); and those browser need a displayer. As a console/command geek, it’s intolerable. After several days investigation, I’d like to introduce PyVirtualDisplay to run Selenium in a console with Python.

## Install in Ubuntu

    sudo apt-get install python-pip
    sudo apt-get install xvfb
    sudo apt-get install xserver-xephyr
    sudo apt-get install tightvncserver
    sudo pip install pyvirtualdisplay
    sudo pip install selenium

## Start selenium in console

    #!/usr/bin/env python
    
    from pyvirtualdisplay import Display
    from selenium import webdriver
    
    display = Display(visible=0, size=(1024, 768))
    display.start()

    driver= webdriver.Firefox()
    actions = webdriver.ActionChains(driver)
    driver.get("http://www.reeline.com/arch/index.html")

    print driver.title

    driver.close()
    display.stop()

## Other purposes

* Increase the PV of a web site
* Cheat in adsense & Baidu union
