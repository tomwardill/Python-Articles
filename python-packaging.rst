Simple guide to Python Packaging
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

Installing an egg
~~~~~~~~~~~~~~~~~

Most python programmers will have come across either `easy_install` or `pip`. This is a simple method of downloading an egg and installing it to your local system::

  >> easy_install Django

By default, these tools will look for the egg on pypi (http://pypi.python.org), which is a egg distribution site and repository that contains most of the publicly released python code in egg form.
A later article will cover using the more advanced installation tool `buildout`.
