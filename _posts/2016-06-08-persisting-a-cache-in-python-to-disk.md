---
layout: post
title: Persisting a Cache in Python to Disk using a decorator
comments: true
date:   2016-06-08 15:00:00
---

Caches are important in helping to solve time complexity issues, and ensure that we don't run a time-consuming program twice. You never know when your scripts can just stop abruptly, and then you lose all the information in your cache, and you have you run everything all over again. 

In order to counter this, saving your cache to a disk is something that can be very helpful in that it allows state to be saved to disk, and be retrieved from it anytime as long as its there.

The function below can be used as a decorator for methods to ensure that the cache is persisted to the disk with a specific filename you can determine to wrap around the original function. 

```python
def persist_cache_to_disk(filename):
    def decorator(original_func):
        try:
            cache = pickle.load(open(filename, 'r'))
        except (IOError, ValueError):
            cache = {}

        atexit.register(lambda: pickle.dump(cache, open(filename, "w")))

        def new_func(*args):
            if tuple(args) not in cache:
                cache[tuple(args)] = original_func(*args)
            return cache[args]

        return new_func

    return decorator
```

You can then use the decorator as such:

```python
@persist_cache_to_disk('users.p')
def get_all_users():
	# your method here
	pass
```


A gist for the code is located here: https://gist.github.com/tohyongcheng/94ca536b0a5c96c9751b82150f20c95a