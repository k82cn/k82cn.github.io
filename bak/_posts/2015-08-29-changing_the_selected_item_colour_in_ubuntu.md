---
layout: post
categories: blogs
title: Changing the Selected Item Colour in Ubuntu 
---

Here is my way on how to change the selected item colour on Ubuntu (14.04); the steps are as follows:

1. To start off, we need to open dconf-editor (Press Alt+F2 and type dconf-editor followed by the enter key).
2. Navigate to the following path: org > gnome > desktop > interface.
3. Find the key called "gtk-color-scheme" and set the value to "selected_bg_color:#023C88" replacing the hex code with the value of the colour you wish to use (I used #ccc for light grey).
4. Done!

