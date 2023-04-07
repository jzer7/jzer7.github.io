---
title: "Python runtime environments in Mac OS"
date: 2022-09-03T16:18:00-05:00
excerpt: "A way to configure and use Python environments, that allows quick experimentation without causing long term pain with among components."
categories:
  - blog
tags:
  - python
  - macos
  - homebrew
---

A few months ago, I ran into a few issues with Python environments at work.
Basically our code needed some libraries, which conflicted with applications bundled in the Linux distribution.
They were both using the same Python runtime, and we could not update one without breaking the other.
So we had to separate our application runtime from that of the Linux distribution.
To avoid causing customer downtime, it took us 2 sprints to fix it! :cry:
Lesson learned: now I am rebuilding my personal development environment.

Other people have suffered similar problems.
Just _google_ "[Python dependency hell](https://www.google.com/search?q=python+dependency+hell)" if you are curious.
{: .notice--info }

The [Twelve-factor app model](https://12factor.net/) asks for the applications to manage its [dependencies](https://12factor.net/dependencies).
Maybe this fits there as well.
{: .notice--info }

## Check the initial state

I use [Homebrew](https://brew.sh/) to manage packages.
When I checked, I saw 3 python runtimes were managed by Homebrew.
```sh
$ brew list|grep -iE 'python'
python@3.10
python@3.9
python@3.8
```

But I have a few others :thinking:
```sh
$ ls -td /usr/local/lib/python3*
/usr/local/lib/python3.10
/usr/local/lib/python3.9
/usr/local/lib/python3.8
/usr/local/lib/python3.7
```
I think they are leftovers from my own experiments.
I will check that later.

To see which packages are using those runtimes:
```sh
$ brew uses --installed python@3.10
awscli     certbot    cffi       docutils   gts        vim
$ brew uses --installed python@3.9
cairo
$ brew uses --installed python@3.8
```
The last one is not used by any packages.

## Resetting the global environment

I had installed some Python modules in those runtime directories (using `pip` outside of a virtual environment), which I need to remove.
And I want to get them back to their natural state.
The first step is to get a backup of my configuration.
```sh
brew bundle dump --all --describe
```
Store the resulting `Brewfile` somewhere safe.
Extra points if you add it to your [dotfiles](https://dotfiles.github.io/).

Now I can start.
I do not need that `python@3.8` so I removed it.
Well, I am not deleting things yet, so I just got it out of the way.
```sh
$ brew remove python@3.8
$ mkdir -p /usr/local/.Attic/lib
$ mv /usr/local/lib/python3.8 /usr/local/.Attic/lib
```

Then I continued with `python@3.9`.
This one is in use, so I am also archiving it, but then I am reinstalling the Homebrew packages that had a use for it.
That will reinstall the Python modules that belong there.
```sh
$ brew uses --installed python@3.9
cairo
$ mv /usr/local/lib/python3.9 /usr/local/.Attic/lib
$ brew reinstall python@3.9 cairo
```

Similarly for the `python@3.10` directory.
```sh
$ brew uses --install python@3.10
awscli     certbot    cffi       docutils   gts        vim
$ brew reinstall python@3.10
$ brew reinstall awscli certbot cffi docutils gts vim
```
I ran some tests to check things were in order.
It is good, but I am keeping that backup for a few months.

## Do it properly

The option that works best for me is to use:
* [pyenv](https://github.com/pyenv/pyenv) to get the basic binaries
* [pipenv](https://docs.pipenv.org/) to create custom environments for each project.

### `pyenv`

`pyenv` is very similar to tools for other environments (e.g., `rbenv` for Ruby, `tfenv` for Terraform).
It gets the runtime from the right source, and creates them in separate places for each.

```sh
brew install pyenv
```

To see which versions I can install:
```sh
pyenv install -l
```

And then install the version(s) of Python I use.
They will be installed under `~/.pyenv/versions`.
```sh
pyenv install 3.9.13
```

But the magic of `pyenv` is how you can quickly switch among them.
This is done by "shims".
Think of them as _wrapper scripts_ around each of the Python binaries, that set the right environment variables depending on which version you want to use.
You can see which shims you have available.
```sh
$ pyenv versions
* system (set by EDITED_DIR/.pyenv/version)
  3.9.13
```

I configured my shell to switch among those.
```sh
cat<<EOM >> ~/.zshrc
## Python Environments
export PYENV_ROOT="$HOME/.pyenv"
eval "$(pyenv init -)"
EOM
```

Now my PATH points to the directory with the _shims_, and those switch the binary accordingly.
I would avoid doing this in production, but it is good in a development environment.

### `pipenv`

`pyenv` takes care of the main binary, but it does not handle the modules I use.
There are tools like [virtualenv](https://virtualenv.pypa.io/) that create directories for your projects.
It is good, and I have used it for years.
But I prefer [pipenv](https://docs.pipenv.org/) for a few reasons:
* `pipenv` integrates naturally with `pyenv`. That results in less tools (e.g., `pyenv-virtualenv`), and less changes to my workflow.
* `pipenv` adds dependency checks and other tooling to manage modules. That means less tools (e.g., `pip-compile`).

The install process is simple:
```sh
brew install pipenv
```

Here is a good [pipenv tutorial](https://realpython.com/pipenv-guide/#example-usage) to get familiar with the usage.

Something nice is that you can create a virtual environment by just indicating the version.
If you do not have the version, it will get it using `pyenv` :heart_eyes:
```sh
$ mkdir -p /tmp/test1
$ cd /tmp/test1
$ pipenv --python 3.7
Warning: Python 3.7 was not found on your system...
Would you like us to install CPython 3.7.16 with Pyenv? [Y/n]:
```

