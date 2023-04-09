---
title: "Google Fire: module for CLI and code exploration"
date: 2022-05-22T15:48:00-05:00
last_modified_at: 2023-04-09T17:07:00-05:00
excerpt: "A module that lets you write a CLI for almost any Python code"
categories:
  - blog
tags:
  - python
---

While looking at some frameworks to write a CLI in Python, I ran into [Google Fire](https://github.com/google/python-fire/) (I hope it's no a pun on Amazon's Fire :wink:).
It is a clever way to interface almost any Python code.

*[CLI]: Command Line Interface

## Basics

Say you have a simple function
```py
def hello(name):
    return 'Hello {name}!'.format(name=name)
```

Import the module and add a call to _Fire_ at the end, and you have a CLI:
```py
if __name__ == '__main__':
    fire.Fire()
```

It will not do much when you run it.
Instead it will display information on how to use it.
```sh
$ python3 test1.py
NAME
    test1.py

SYNOPSIS
    test1.py GROUP | COMMAND

GROUPS
    GROUP is one of the following:

     fire
       The Python Fire module.

COMMANDS
    COMMAND is one of the following:

     hello
```

Now you know there is 1 valid command (the function `hello`).
So to call it, we specify an argument with that name, and any arguments the function expects.
```sh
$ python3 test1.py hello Universe
Hello Universe!
```

That seemed more complex than what I want.
After all, now I need to pass an extra argument when I call my script.
You can avoid that, by modifying the call to _Fire_ to be:
```py
if __name__ == '__main__':
    fire.Fire(hello)
```

Now _Fire_ passes all arguments directly to that function.
```sh
$ python3 test1.py Ocean
Hello Ocean!
```

## More

But this model is very extensible when we have more than 1 function.
On top of that it also works with classes and modules.
For example:
```py
import fire

"""A test of Fire with a class Object"""


class Calculator(object):
    """A simple calculator class."""

    def add(self, number1, number2):
        """Add two numbers together"""
        return number1 + number2

    def multiple(self, number1, number2):
        """Multiply two numbers together"""
        return number1 * number2


if __name__ == '__main__':
    fire.Fire(Calculator)
```

And then, you can use it like:
```sh
$ python3 test2.py add 2.04 1.1
3.14
```

Oh, and it reads [Python docstrings](https://realpython.com/documenting-python-code/#documenting-your-python-code-base-using-docstrings) to get you proper help. :heart_eyes:
```sh
$ python3 test2.py
NAME
    test2.py - A simple calculator class.

SYNOPSIS
    test2.py COMMAND

DESCRIPTION
    A simple calculator class.

COMMANDS
    COMMAND is one of the following:

     add
       Add two numbers together

     multiple
       Multiply two numbers together
```
