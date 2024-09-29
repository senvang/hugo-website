---
title: Theme update
date: 2024-09-30T02:19:00+07:00
author: sroemer
categories:
- it
tags:
- website
- hugo
- theme
- terminal
keywords:
- website
- hugo
- theme
- terminal
showFullContent: true
readingTime: false
hideComments: false
---

I use the static site generator [Hugo](https://gohugo.io/) to create this website and chose the
[Terminal](https://github.com/panr/hugo-theme-terminal) theme originally created by the Github
user [panr](https://github.com/panr). Here I use my own [branch](https://github.com/senvang/hugo-theme-terminal) which replaces the font `FiraCode` with `FiraMono` to get rid of the ligatures.

Today I found out that `panr` continued development on his theme after he actually stopped a
while ago. A happy surprise! So I just merged his latest version into my branch and updated
the website. He actually introduced some major changes by now relying on his new project
[Terminal.css](https://github.com/panr/terminal-css) which simply separates the style from
the theme to allow people to use it independently.

This now also easily allows the [creation](https://panr.github.io/terminal-css/) of custom
color schemes (compared to having just a set of fixed colors before). I used this to slightly
adapt the colors to my company logo.

Besides this there should be no changes here. Thanks `panr` ... 
