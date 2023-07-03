Uninstall standard Apple Python distribution
============================================

remove the Python 2.7 framework::

    sudo rm -rf /Library/Frameworks/Python.framework/Versions/2.7


remove the Python 2.7 applications directory::

    sudo rm -rf "/Applications/Python 2.7"

remove the symbolic links in /usr/local/bin that point to this python version see ls -l /usr/local/bin

    cd /usr/local/bin; 
    ls -l . | grep '../Library/Frameworks/Python.framework/Versions/2.7' | awk '{print $9}' | xargs rm


Select python from port installation
====================================

To list python installations::

    port select --list python


To show current selected version::

    port select --show python


To select::

    sudo port select --set python <the python version>
    
    
Install libraries needed for compiling PIL
==========================================

Jpeg::

    sudo port install jpeg
    
Freetype::

    sudo port install freetype

port install libyaml
port install postgresql84

Install buildout that uses macport libraries
============================================

::

    sudo port install py27-zc-buildout


Bootstrap installation::

    buildout-2.7 bootstrap
    bin/buildout


Install OpenOffice modules
==========================

No module named uno
No module named unohelper
No module named com.sun.star.beans
No module named com.sun.star.uno
No module named com.sun.star.connection
No module named com.sun.star.beans
No module named com.sun.star.lang
No module named com.sun.star.io
No module named com.sun.star.io