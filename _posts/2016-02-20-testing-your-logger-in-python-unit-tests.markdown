---
layout: post
comments: true
title:  "Testing your logger in Python unit tests"
date:   2016-02-20 11:00:00
categories: "python"
---

When in developing in large scale applications, we definitely need to do proper logging in order to know where all our errors are happening at.

In a Django project, I was trying to find ways to test whether my logger had been called to execute a logging action. In my case, you may want to test whether a log message is created when an exception is raised. This is going to be easy, and what you will need to use is just Python's Mock class to mock the logger in the method.

Let's say we have a simple function in python to get a value from a dictionary:

{% highlight python %}
import logging

def get_value_from_dict(dict, key):
    try:
        return dict[key]
    except KeyError:
        logging.warn("The dictionary does not have a key value of %s" % key)
    return None

{% endhighlight %}

Now, in order to test if the function executes `logging.warn`, we can just run the following testcase:

{% highlight python %}

from unittest import TestCase
from my_own_module import get_value_from_dict
from my_own_module import logging

class MyUnitTestCase(TestCase):
    
    @mock.patch(logging)
    def test_get_value_from_dict_has_logging(self, mock_logger):
        d = {}
        key = 1
        get_value_from_dict(d, key)
        mock_logger.warn.assert_called_with("The dictionary does not have a key value of %s" % key)

{% endhighlight %}

With this, we are able to test whether `warn` is called, and if it was called with the right argument. You might ask, where did `assert_called_with` come from? It actually comes from the mock object that we created from `@mock.patch(logging)`. We have patched `logging` with a Mock object that comes with the following assertion methods:

{% highlight python %}

assert_called_with(*args, **kwargs) # assert that calls are made in a particular way
assert_called_once_with(*args, **kwargs) # asserts that method is called once with specified arguments
assert_any_call(*args, **kwargs) # assert the mock has been called with the specified arguments.
assert_has_calls(calls, any_order=False) # assert the mock has been called with the specified calls.
assert_not_called(*args, **kwargs) # assert the mock was never called.

{% endhighlight %}

There are even more methods that you can test out with, and they are all in the docs [here](https://docs.python.org/2/library/logging.html). You can now test your logger with any of these methods to ensure that your logic is right!

