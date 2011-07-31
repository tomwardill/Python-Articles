Buildout 101
============

Introduction to the series
--------------------------

This is the first in a 3 part series of tutorials about creating, configuring and maintaining buildout configuration files, and how buildout can be used to deploy and configure both python-based and other software.
During the course of this series, I will cover buildout configuration files, some buildout recipes and a simple overview structure of a buildout recipe. I will not cover creating a recipe, or developing buildout itself.

All code samples will be python 2.4+ compatible, system command lines will be debian/ubuntu specific, but simple enough to generalise out to most systems (OSX and Windows included).
Where a sample project or code is required, I've used Django as it's what I'm most familiar with, but this series is all about the techniques and configuration in buildout itself, it is **not** Django specific, so don't be scared off if you happen to be using something else.

Buildout Basics
===============

So, what's this buildout thing anyway?
--------------------------------------

If you're a python developer, or even just a python user, you will probably have come across either ``easy_install`` or ``pip`` (or both). These are pretty much two methods of achieving the same thing; namely, installing python software ::

$> sudo easy_install Django

This is a fairly simple command, it will install Django onto the system path, so from anywhere you can do ::

>>> import django
>>>

While this is handy, it's not ideal for production deployment. System installing a package will lead to problems with maintenance, and probably also lead to version conflict problems, particuarly if you have multiple sites or environments deployed on the same server. One environment may need Django 1.1, the newest may need 1.3. There are significant differences in the framework from one major version to another, and a clean upgrade may not be possible. So, system-installing things is generally considered to be a bad idea, unless it's guaranteed to be a dedicated machine.

So what do you do about it? 
One answer is to use a ``virtualenv``. This is a python package that will create a 'clean' python environment in a particular directory::

$> virtualenv deployment --no-site-packages

This will create a directory called 'deployment', in which is a clean python intepreter with only a local path. This environment will ignore any system-installed packages (``--no-site-packages``), and give you a fresh, self contained python environment.

Once you have activated this environment, you can then easy_install or pip install the packages you require, and then use them as you would normally, safe in the knowledge that anything you install in this environment is not going to affect the mission-critical sales (or pie-ordering website) process that's running in the directory next door.

So, if virtualenv can solve this problem for us, why do we need something else, something more complex to solve essentially the same problem?

The answer, is that buildout doesn't just solve this problem, it solves a whole lot more problems, particuarly when it comes to 'how do I release this code to production, yet make sure I can still work on it, without breaking the release?'

Building something with buildout
--------------------------------

The intent is to show you the parts for a buildout config, then show how it all fits together. If you want to see the final product, then dissemble it to find the overall picture, scroll to the end of this, have a look, then come back. Go on, it's digital, this will still be here when you come back....

Config Files
~~~~~~~~~~~~

Pretty much everything that happens with buildout is controlled by its config file (this isn't quite true, but hey, 'basics'). A config file is a simple ini (``ConfigParser``) style text file; that defines some sections, some options for those sections, and some choices in the options.

In this case, the sections of a buildout config file (henceforth referred to as ``buildout.cfg``) are generally referred to as ``parts``. The most important of these parts is the ``buildout`` part itself, which controls the options for the buildout process.

An absolute minimum buildout part looks something like this::

 [buildout]
 parts = noseinstall

While this is not a complete buildout.cfg, it is the minimum that is required in the buildout part itself. All is is doing is listing the other parts that buildout will use to actually do something, in this case, it is looking for a single part named ``noseinstall``. As this part doesn't exist yet, it won't actually work. So, lets add the part, and in the next section, see what it does::

 [buildout]
 parts = noseinstall

 [noseinstall]
 recipe = zc.recipe.egg
 eggs = Nose

An aside about bootstrap.py
~~~~~~~~~~~~~~~~~~~~~~~~~~~

We now have a config file that we're reasonably sure will do something, if we're really lucky, it'll do something that we actually want it to do. But how do we run it? We will need buildout itself, but we don't have that yet. At this point, there are two ways to proceed.

1. ``sudo apt-get install python-zc.buildout``
2. ``wget http://python-distribute.org/bootstrap.py && python bootstrap.py``

For various reasons, unless you need a very specific version of buildout, it is best to use the bootstrap.py file. This is a simple file that contains enough of buildout to install buildout itself inside your environment. As it's cleanly installed, it can be upgraded, version pinned and generally used in the same manner as in a virtualenv style build. If you system-install buildout, you will not be able to easily upgrade the buildout instance, and may run into version conflicts if a project specifies a version newer than the one you have. Both approaches have their advantages, I prefer the second as it is slightly more self contained. Mixing the approaches (using bootstrap.py with a system-install is possible, but can expose some bugs in the buildout install procedure).

The rest of this document is going to assume that you have used bootstrap.py to install buildout.

Running some buildout
~~~~~~~~~~~~~~~~~~~~~

Now we have a method of running buildout, it's time to do it in the directory where we left the buildout.cfg file created earlier::

 $> bin/buildout

At this point, buildout will output something along the lines of::
  
  Getting distribution for 'zc.recipe.egg'.
  Got zc.recipe.egg 1.3.2.
  Installing noseinstall.
  Getting distribution for 'Nose'.
  no previously-included directories found matching 'doc/.build'
  Got nose 1.0.0.
  Generated script '/home/tomwardill/tmp/buildoutwriteup/bin/nosetests-2.6'.
  Generated script '/home/tomwardill/tmp/buildoutwriteup/bin/nosetests'.

Your output may not be exactly similar, but should contain broadly those lines.

The simple sample here is using the zc.recipe.egg recipe. This is probably the most common of all buildout recipes as it is the one that will do the heavy work of downloading an egg, analysing its setup.py for dependencies (and installing them if required), and then finally installing the egg into the buildout path for use. Recipes are just python eggs that contain code that buildout will run. The easiest way to think of this is that while a recipe is an egg, recipe contains instructions for the buildout process itself, and therefore will not be available to code at the end.

An analysis of the buildout output shows exactly what it has done. It has downloaded an egg for ``zc.recipe.egg`` and run the ``noseinstall`` part. Let's take a closer look at that noseinstall part from before::

 [noseinstall]
 recipe = zc.recipe.egg
 eggs = Nose

So, we can see why buildout has installed zc.recipe.egg, it is specified in the ``recipe`` option of this part, so buildout will download it, install it and then run it. We will take a closer look at the construction of a recipe in a later article, but for now, assume that buildout has executed a bunch of python code in the recipe, and we'll carry on.
The python code in this case will look at the part that it is in, and look for an option called ``eggs``. As we have specified this option, it will then look at this as a list, and install all the eggs that we have listed; in this case, just the one, the unittest test runner Nose.
As you can see from the bottom of the buildout output, the recipe has downloaded Nose, extracted it and created two files; ``bin/nosetests`` and ``bin/nosetests-2.6``. Running one of those files like so::

 $> bin/nosetests

 ----------------------------------------------------------------------
 Ran 0 tests in 0.002s

 OK
 $>

We can see that this is nose, as we expect it to be. Two files have been generated because that is that the setup.py for Nose defines, a base nosetest executable, and one for the specifc python version that we have used (python 2.6 in my case). These are specified in the setup.py that makes up the nose egg, which will be covered in a later article.

Conclusion
----------

We can install buildout into a development environment, and use a simple config file to install a python egg. The next article will cover blahdy blah, with some extra blah.
