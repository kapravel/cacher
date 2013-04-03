cacher
======

Awesome caching module for python.

# Usage

- import the cacher:

```python
import cacher
```

- wrap a function

```python
@cacher.wrap()
def some_long_function(n):
  pass
```

- Call the function!

```python
some_long_function(1) # takes forever!!!
some_long_function(1) # super fast!!!
some_long_function(2) # takes forever!!!
some_long_function(2) # super fast!!!
```

- If needed, you can get at the original function by doing:

```python
some_long_function.wrapped_func(1) # takes forever!!!
```

# Options

The wrap function takes a number of arguments to build the cache filename. They are:

- dir_name - the directory (under the current subdir) to put the file in. Default: ''
- module_name - the name of the module. Default: function.__module__
- function_name - the name of the function. Default: function.__name__
- version_name - the version of the function. Default: str(hash(f.__code__))
- each - if this is True, the arguments to the function are cached and included in the filename. Default: False

It also takes other options to determine how and when caches are saved:

- disk - save the cache out to disk. Default: True
- autosave - autosave the cache to disk after each added value. Useless without disk. Default: False
- autodiscard - discard the cache after every use. Useful with 'each' and big cached things. Default: False

The autosave and autodiscard options might do unexpected things, especially when recursion is concerned, so be careful!

# Advanced features

## helper functions
The wrapped function will have several useful attributes:

- cache_orig - the original (unwrapped function) for direct access
- cache_dict - returns the cache dictionary. Note that this doesn't work well with autosave and autodiscard.
- cache_check - checks whether a set of arguments are in the cache
- cache_setter - returns a setter function for specific arguments that can be used set the cache
- cache_remove - removes an entry from the cache
- cache_discard - discards the cache (allowing it to be reloaded at the next call)
- cache_clear - clears the cache (replaces it with an empty dictionary)
- cache_save - saves the cache to disk
- cache_load - loads the cache from disk

Here are some examples:

```python
@cacher.wrap(version_name = "something", module_name = "woo")
def zomg(a):
  return 1

zomg(1) # of course, return value is 1

zomg.cache_setter(1)(10)
zomg(1) # returns 10
zomg.cache_orig(1) # returns 1
zomg(1) # returns 10

zomg.cache_discard()
zomg(1) # return value is 1

## cache subdirectories

Sometimes, for various reasons, you might want to have seperate unrelated caches for the same project. This is facilitated by subdirectories:

```python
@cacher.wrap(version_name = "something", module_name = "woo")
def zomg(a):
  pass

zomg(1) # would be cached in cached/woo.zomg%something.p

cacher.push_subdir("sub1") # the dirstack would be "" and the current dir would be "sub1"
zomg(1) # would be cached in cached/sub1/woo.zomg%something.p

cacher.push_subdir("sub2") # the dirstack would be "", "sub1" and the current dir would be "sub2"
zomg(1) # would be cached in cached/sub2/woo.zomg%something.p

cacher.pop_subdir() # the dirstack would be "" and the current dir would be "sub1"
zomg(1) # would be cached in cached/sub1/woo.zomg%something.p

cacher.set_subdir("sub3") # the dirstack would be "" and the current dir would be "sub3"
zomg(1) # would be cached in cached/sub3/woo.zomg%something.p

cacher.pop_subdir()
zomg(1) # would be cached in cached/woo.zomg%something.p
```

Every time a directory change takes place, the cacher flushes all the caches that have changed (and are not set to disk=False) out to disk.

## logging

The cacher logs using the "cacher" logger. You can turn on debugging by doing:

```python
import logging
logging.getLogger("cacher").setLevel(logging.DEBUG)
```

# Bugs

- no easy class method support
- currently, the key for the arguments is generated by doing "(str(args), str(kwargs))". This has several problems:
	- things such as sets don't always str() to the same string in terms of order
	- it's slow
