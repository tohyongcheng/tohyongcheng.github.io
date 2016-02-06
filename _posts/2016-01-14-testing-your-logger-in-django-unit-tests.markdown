---
layout: post
comments: true
title:  "Testing your logger in Django unit tests"
date:   2016-01-14 18:00:00
categories: django
published: false
---

While working on a Django project, I was trying to find ways to test whether my logger had been called to execute a logging action. In my case, you may want to test whether a log message is created when an exception is raised. This is going to be easy, and what you will need to use is just Python's Mock class to mock the logger in the method.

