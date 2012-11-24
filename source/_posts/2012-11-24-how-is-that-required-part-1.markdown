---
layout: post
title: "How the hell is that required? Part 1"
date: 2012-11-24 01:33
comments: true
categories: 
---
When I dive into the codebase of an unfamiliar Ruby project, divining exactly
how the project includes source files frequently trips me up. Where are they
modifying the path? Do I need to use `require_relative`? Is the framework
reloading my code? How are the tests isolating my code? Wait, what is Ruby's
default include path again?

In part one of this series on Ruby source organization, I’m going to provide
a primer on Ruby’s basic code loading mechanisms.

## How do I load thee? Let me count the ways. Five.
Ruby 1.9.3 provides five mechanisms to load a source file. In each case, the
file extension may be omitted if it is `.rb`, `.so`, or `.dll`.

### 1. `require_relative ‘../lib/my_source’`
The `require_relative` message on the `Kernel` object is the simplest way to
load code. It is given a file path relative to **the current source file**, and
proceeds to load the source file at that location. You don’t need any
additional information to figure out which file is loaded.

### 2. `require ‘my_source’`
The `require` message on the `Kernel` object operates like `require_relative`,
but does not accept paths relative to the current source file. Instead, it
accepts either an absolute path (e.g.
`’/Users/testuser/home/testproject/my_source’`) or a path relative to the
folders in the VM’s `$LOAD_PATH` global variable (e.g. `’my_source’`). We’ll go
over the `$LOAD_PATH` variable later.

### 3. `> ruby -r my_source runner.rb`
The `ruby` command line executable accepts a `-r` option that requires a source
file equivalently to the `require ‘my_source’` code above.

### 4. `load ‘my_source’`
The `load` message on the `Kernel` object operates like `require`, with
a couple of differences. First, it will not propagate any local variables in
the loaded file into the global namespace. Second, it will attempt to load and
run a source file every time it is sent (while all of the `require` methods
only load a file once.) And finally, it can be given a second optional argument
of `true` to sandbox the loaded script within an anonymous module.

### 5. `autoload :MyClass, ‘my_source’`
The `autoload` message on the `Kernel` object accepts a module name (including
class names) as either a symbol or a string, as well as a path. It tells the VM
to `Kernel#require` the specified path the first time that the given module is
accessed.

## What’s in a `$LOAD_PATH`?
With the exception of `require_relative`, all of these mechanisms can either
accept an absolute path or a path relative to the directories specified in the
global `$LOAD_PATH` variable (which can also be accessed as `$:`. The variable
is simply an array of absolute paths and paths **relative to the current
directory**.  It will by default include the standard library paths and the
`lib/` folders of any installed gems. It will **not** include the current
directory, which can seem a little counter-intuitive at first. So how do we
modify the load path to include the directories we want? Here are two common
ways:

### 1. `> ruby -I lib runner.rb`
The `ruby`  command line executable accepts a `-I` option that includes the
given directory in the `$LOAD_PATH`. The directory path given may be absolute
or **relative to the current directory**.

### 2. `$LOAD_PATH << “lib”`
The `$LOAD_PATH` may also be modified directly by the application. We just need
to insert a new directory path into the array. The `Object#__FILE__` message
provides a path to the currently executing file, which gives us a way to build
new relative directory paths:
``` ruby
# create a new absolute path by appending ‘lib’ to the current file’s directory
new_path = File.join(File.expand_path(File.dirname(__FILE__) ), “lib”)
# add the new path to the load path
$LOAD_PATH << new_path
```

## Up next
That's it for Ruby's built-in code loading mechanics. In part two of this
series, I'll explore common project organization patterns and idioms.
