A Guide to Python Packaging
================================

Introduction
------------

This article is intended to serve as a simple explanation of python packaging, including why you should package your code correctly, and how to release it to the world. It is not intended to be an in-depth or complete guide to the different ways python can be distributed, or the many options for the setup.py.
It is hoped that, having read this, you will be able to layout, setup, package and upload your code to pypi or another distribution source.


Why do we need to package? When do we do it? What's wrong with a source checkout anyway?

Why do we need to package?
~~~~~~~~~~~~~~~~~~~~~~~~~~

Containing your code in a reusable module is a fundamental part of programming, it allows for easy maintenance, use and distribution. Packaging code properly is related to this, having a clean module on your computer is okay, but then, how do you ship that to a server, or another person?
Python packaging is designed to solve this problem, by allowing you to bundle up your code and associated files into a single, easy to distribute form.
Packaging correctly will also allow you to specify dependencies for your code, automated tools for installing these exist; again reducing the cost of shipping your code.

When do we do it?
~~~~~~~~~~~~~~~~~

There is no easy answer to this question. If it's a quick hack on your machine to test something, then the overhead is not worth the effort. If it's production code, or for public release, then it's definitely worth getting right at the start, rather than having to retrofit afterwards. A good start will help you easily release code, as the time/effort cost is negligible if you've started in the correct manner.

From a personal perspective, as soon as the code I am writing has exceed one file, or has started to rely on dependencies that are not usually available, I will take the time to package the code in some manner.

What's wrong with a source checkout anyway?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

I'm hoping that the two questions above have given you enough information to answer this question. A source checkout is an easy method of distribution, but will not allow for automated dependency installation, or easy and automated upgrading and package maintenance.

Eggs
----

The chosen method of python packaging is called 'Eggs' (as pythons lay eggs. Don't look at me, I'm not responsible for the name.). Eggs are a compressed archive (tarball or zip file) of the python code and accompanying files, with some metadata embedded into them to describe the structure and names.
If you are familiar with linux systems, eggs are roughly correspondent to a .deb or .rpm, with a similar ecosystem around the installation and maintenance.

An aside about setuptools
~~~~~~~~~~~~~~~~~~~~~~~~~

The core mechanisms of python packaging live in a module called `setuptools`. This ships with most python distributions, and is included in python 3 and above. There are a few legacy versions of setuptools (which are mostly compatible, and mostly differ only on implementation details), the current method of choice is `distribute` (or `distutils2` for python3).
As you will see below, setuptools is the main engine that enables both the creation of python packages, and the installation and maintenance of the same.

Installing an egg
~~~~~~~~~~~~~~~~~

Most python programmers will have come across either `easy_install` or `pip`. This is a simple method of downloading an egg and installing it to your local system::

  >> easy_install Django

By default, these tools will look for the egg on pypi (http://pypi.python.org), which is a egg distribution site and repository that contains most of the publicly released python code in egg form.
A later article will cover using the more advanced installation tool `buildout`.

Components of a Egg
-------------------

Most python eggs has 3 basic components:

 1. A `setup.py`
 2. Some code
 3. Accompanying documents

setup.py
~~~~~~~~

The setup.py is the main engine of an egg, it contains the metadata, package and namespace information that will determine how the egg behaves, and what is packaged into the archive when they are created.
At it's core, the setup.py is simply a call to a python method from setuptools (or distribute), with the parameters to the method being the information relevant to that package.

There is a large list of available parameters, some of which are required to make a valid egg, and others that are very rarely, if ever used. For the full list of options, see the python documentation on making eggs << insert link here >>

A basic setup.py will look something like this::

    from setuptools import setup, find_packages

	setup(
	    name = 'a sample egg',
	    version = '0.0.1',
	    description = "a short description",
	    long_description = "a longer description of what this egg does",
	    author = "Tom Wardill",
	    author_email = "tom@howrandom.net",
	    url = "http://github.com/tomwardill",
	    license = "GPL",
	    packages = find_packages(),
	    include_package_data = True,
	    zip_safe = False,
	    )

Some of this is reasonably obvious what it does, others involve various levels of setuptools magic, and will not be immediately obvious, particularly `find_packages` and `include_package_data`. This basic skeleton with another couple of files will suffice to package an egg from your code.

Package Layout
~~~~~~~~~~~~~~

There are many different ways to layout the file structure of your package. In this section, I have chosen a simple method that will work in most situations, with the default options in setuptools.

Assume that you have a python module named `robopicker`. This module consists of 3 python files (`first.py`, `second.py` and `third.py`), and a data file (`data.json`).

I would lay this module out on disk like this::

  > setup.py
  > README.rst
  > MANIFEST.in
  > robopicker/
    > first.py
    > second.py
    > third.py
    > data/
      > data.json

This layout gives a clear indication as to what is your code, and what is metadata about the package that will contain your code. There are other ways to do it that are valid, I happen to prefer this one.

`MANIFEST.in` and `include_package_data`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These two things are related. MANIFEST.in is a text file containing descriptions of what none-code files you would like to include in your egg. 
`include_package_data` is the parameter to the `setup` method in `setup.py` which tells setuptools to include extraneous data into the egg.

The simplest way to include none-code files into your egg is to set `include_package_data` to `True` and specify the files that you want in the `MANIFEST.in`.

Using the sample `setup.py` from above, this `MANIFEST.in` would be enough to include the files that we want::

  include README.rst
  recursive-include robopicker/data/*

As can be seen, file globbing and wildcards are supported by the `MANIFEST.in`, and the command structure is fairly simple. For a more detailed explanation of all the available options, see the official documentation (http://docs.python.org/distutils/sourcedist.html#the-manifest-in-template)