Upgrading from Lion to Mountain Lion - from the eyes of a Server Density Engineer.
==================================================================================

Everyone wants to update to the new shiny, so this is a quick tale and description of what I did, what went wrong, and how to fix it.

Before Upgrade
--------------

Take a backup of your `/etc/apache2/httpd.conf` and `/etc/php.ini`, or their equivalents if you run from different files.

Upgrade Process
---------------

The upgrade process went pretty smoothly, download from the App Store, then run the installer. Only thing to watch for is you have to enter your password earlier than expected if you have FileVault activated (on the first reboot, don't leave it sat there for ages unattended....).

Apps after the upgrade
----------------------

The only thing I've found so far that doesn't work very well is ScanDrop, but that's fairly minor.
PHPStorm, iTerm, Sublime Text 2, and the other apps I spend my days in seem to be working so far.
WingIDE required that I install [XQuartz](http://www.xquartz.org), but that was straightforward and expected.

Python
------

This is where some problems occured.

1. Anything `easy_install` or `pip` installed no longer works. The python version has changed, so the site-packages folder has changed and is empty. Reinstall them, assuming you don't have the problems below.
2. `pip` fails with `distribute not found`. There is a system-wide extra package directory that has some packages installed in it (I assume they are for OSX itself). It seems to be that whatever process cleans this out leaves the `.pyc` files behind.
Not a massive problem in itself, but one of the files is the easy_install.pyc, which contains a broken system path. ```sudo rm /System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/*.pyc``` will sort that out.
3. Once this is done, you might need to reinstall `pip` or `distribute` from their relevant sites.

This process should fix any buildouts you have, though you might need to rerun buildout if you have anything that is version sensitive, or has native compilation.

Homebrew
--------

I can't think of anything that specifically broke in the upgrade, most Formulae have been updated by now. `brew update && brew upgrade` took care of everything that I needed.

Apache
------

The upgrade will replace your httpd.conf with a default copy. I assume there are differences in it, but as I didn't have a copy of my original one (see the first section, learn from my experience...), I can't diff them.
I had to:

1. Re-enable `php`
2. Re-enable the import of `vhosts.conf`

Then it started working.

PHP
---

Again, the upgrade replaced/removed my php.ini. Adding the settings back to the end of it fixed it. The settings I've used are included below, this might change per your environment

	[xdebug]
	zend_extension="/usr/local/Cellar/php53-xdebug/2.2.1/xdebug.so"
	xdebug.remote_enable=1
	xdebug.remote_host=localhost
	xdebug.remote_port=9000
	xdebug.remote_autostart=1

	[mongo]
	extension="/usr/local/Cellar/php53-mongo/1.2.11/mongo.so"

	[mcrypt]
	extension="/usr/local/Cellar/php53-mcrypt/5.3.13/mcrypt.so"

XDebug
------

This was a strange one. Apple now ship a built version of XDebug in OSX. This has the path in the `httpd.conf` by default, though it's disabled. Enabling this copy lead to some strange behaviour.

If I just had it enabled, everything was fine. If I attached a listener (PHPStorm in this case) to the XDebug instance, the pages stopped rendering. Chrome thought that the data was being recieved, but the page would be blank, or the AJAX request would fail.

In the end, switching to the homebrew installed xdebug (or rebuilding that if it's already installed) fixed the problem.

No idea as to the cause, might be a local thing, but if you see it; I hope that's an answer for you.

The End
-------

That's all I've found so far, if I find anymore, I'll update this post.
