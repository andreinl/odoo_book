OpenERP installation
********************

Installation in virtualenv v.8.0
================================

.. note:: Tested on Mac OS X

Create PostgerSQL user. From PostgreSQL prompt::

    =# CREATE USER openerp SUPERUSER WITH PASSWORD '123456';
    =# CREATE ROLE openerp SUPERUSER PASSWORD '123456';
    =# ALTER ROLE openerp with LOGIN;


Create virtualenv, then::

    $ workon openerp_80
    $ git clone https://github.com/OCA/OCB.git Odoo_80_ocb
    $ cd Odoo_80
    $ export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/<your version>/bin
    $ pip install -r requirements.txt
    $ hg clone ssh://mercurial@46.30.244.161//home/mercurial/didotech_80 didotech_80
    $ cd ../addons_80
    $ git clone https://github.com/OCA/server-tools.git
    $ git clone https://github.com/OCA/partner-contact.git
    $ git clone https://github.com/OCA/l10n-italy.git
    

Installation in virtualenv v.7.0
================================

First create virtualenv::

    $ mkdir Openerp_7_ocb
    $ cd
    $ cd .virtualenvs
    $ virtualenv --system-site-packages virtual_erp_7
    $ cd Openerp_7_ocb/

Then install by hand PyChart and aeroolib::
    
    (virtual_erp_61)/src$ wget http://download.gna.org/pychart/PyChart-1.39.tar.gz
    (virtual_erp_61)/src$ tar xvfz PyChart-1.39.tar.gz   
    (virtual_erp_61)/src$ pip install -e PyChart-1.39

Install python libraries con pip::
    
    $ pip install mock
    $ pip install python-openid
    $ pip install psycopg2
    $ pip install psutil
    $ pip install Flask-Babel
    $ pip install unittest2
    $ pip install vatnumber
    $ pip install vobject
    $ pip install pywebdav
    $ pip install xlwt
    $ pip install xlrd
    $ pip install zsi
    $ pip install genshi==0.6
    
Or you can create file requierements.txt::

    mock==1.0.1
    python-openid==2.2.5
    psycopg2==2.5
    psutil==0.7.1
    Flask-Babel==0.8
    unittest2==0.5.1
    vatnumber==1.1
    vobject==0.8.1c
    PyWebDAV==0.9.8
    xlwt==0.7.5
    xlrd==0.9.2
    ZSI==2.0-rc3
    Genshi==0.6
    
    ## This packets I find already present on my machine
    python-dateutil==1.5
    docutils==0.8.1
    feedparser==5.1
    gdata==2.0.14
    Jinja2==2.6
    
    lxml==2.3.2
    Mako==0.5.0
    pydot==1.0.2
    pyparsing==1.5.2
    reportlab==2.5
    simplejson==2.3.2
    pytz==2011k
    Werkzeug==0.9.1
    PyYAML==3.10


This packets are also present on my machine, but freese don't write their version::
    
    python-ldap
    python-libxslt1


.. note:: ``pip freeze > requirements.txt`` produce file with all libraries accessible in this environment.

Install Aeroo Reports::

    (virtual_erp_7)/src$ wget https://launchpad.net/aeroolib/trunk/1.0.0/+download/aeroolib.tar.gz
    (virtual_erp_7)/src$ tar xvfz aeroolib.tar.gz
    (virtual_erp_7)/src$ pip install -e aeroolib


Now download and install server, addons and aeroo::

    (virtual_erp_7)$ bzr branch lp:ocb-server openerp-server
    (virtual_erp_7)$ bzr branch lp:ocb-web openerp-web
    (virtual_erp_7)$ bzr branch lp:ocb-addons addons-ocb
    (virtual_erp_7)$ bzr branch lp:aeroo aeroo


Create *openerp.cfg* and put it in *etc/* directory.

Create a launcher *start_openerp.sh* and put it in *bin/* directory:

.. code-block:: guess

    #!/bin/bash

    ENV=".virtualenvs/virtual_erp_7"
    args=("$@")

    cd ~/$ENV
    . bin/activate

    cd /dati/lp/Openerp_7_ocb/openerp-server/
    echo "Args ${#args[@]}"

    if [ ${#args[@]} -gt 0 ]; then
        echo "./openerp-server --update=$1"
        ./openerp-server --update=$1 -c ../etc/openerp.cfg
    else
        reset
        ./openerp-server -c ../etc/openerp.cfg
    fi


Installation in virtualenv v.6.1
================================

First create virtualenv::

    $ mkdir Virtual_Openerp_61_ocb
    $ cd
    $ cd .virtualenvs
    $ virtualenv --system-site-packages virtual_erp_61
    $ cd Virtual_Openerp_61_ocb/


.. note:: don't use $ mkvirtualenv virtual_erp_61 as this will create completely isolated environment, but we do need some libraries (like pycups) that is difficult to install in isolated environment


Then install by hand PyChart and aeroolib::
    
    (virtual_erp_61)/src$ wget http://download.gna.org/pychart/PyChart-1.39.tar.gz
    (virtual_erp_61)/src$ tar xvfz PyChart-1.39.tar.gz   
    (virtual_erp_61)/src$ pip install -e PyChart-1.39
    

Create file requirements.txt::

    MarkupSafe==0.15
    Pillow==1.7.7
    PyXML==0.8.4
    babel==0.9.6
    feedparser==5.1.1
    gdata==2.0.16
    lxml==2.3.3
    mako==0.6.2
    psycopg2==2.4.4
    pydot==1.0.28
    pyparsing==1.5.6
    python-dateutil==1.5
    python-ldap==2.4.10
    python-openid==2.2.5
    pytz==2012b
    pywebdav==0.9.4.1
    pyyaml==3.10
    reportlab==2.5
    simplejson==2.4.0
    vatnumber==1.0
    vobject==0.8.1c
    werkzeug==0.8.3
    xlwt==0.7.3
    zsi==2.0-rc3
    genshi==0.6


Then download and install everything with command::

    (virtual_erp_61)/src$ pip install -r requirements.txt
    (virtual_erp_61)/src$ wget https://launchpad.net/aeroolib/trunk/1.0.0/+download/aeroolib.tar.gz
    (virtual_erp_61)/src$ tar xvfz aeroolib.tar.gz
    (virtual_erp_61)/src$ pip install -e aeroolib

Now download and install server, addons and aeroo::

    (virtual_erp_61)$ bzr branch lp:ocb-server/6.1 openerp61-server
    (virtual_erp_61)$ bzr branch lp:ocb-web/6.1 openerp-web
    (virtual_erp_61)$ bzr branch lp:ocb-addons/6.1 addons-ocb
    (virtual_erp_61)$ bzr branch lp:aeroo/openerp6.1.x aeroo



Installation with buildout
==========================

The most simple way to install OpenERP is with buildout.

First we should install some missing libraries. Type as root::

    # pip install zc.buildout
    # apt-get install libldap2-dev
    # apt-get install libsasl2-dev
    # apt-get install libssl-dev


As normal user (not root)::

    $ mkvirtualenv OpenERP_6.1
    (OpenERP_6.1)$ mkdir OpenERP_6.1
    (OpenERP_6.1)$ cd OpenERP_6.1

Create file *buildout.cfg*

.. note:: 
    This is work in progress. At the moment I use this one, but may be there are better variants.
    
::

    [buildout]
    parts = openerp
    versions = versions
    find-links = http://download.gna.org/pychart/


    [openerp]
    recipe = anybox.recipe.openerp:server
    version = bzr lp:ocb-server/6.1 openerp61-server last:1
    addons = local /dati/lp/didotech/openerp_6.1
             bzr lp:ocb-web/6.1 openerp-web last:1 subdir=addons
             bzr lp:ocb-addons/6.1 addons-ocb last:1
             bzr lp:aeroo/openerp6.1.x aeroo last:1
    
    options.admin_passwd = admin
    options.db_user = openerp
    options.db_password = xxxxxx
    options.log_handler = [':DEBUG']

    [aeroolib]
    recipe = gocept.download
    url = https://launchpad.net/aeroolib/trunk/1.0.0/+download/aeroolib.tar.gz
    strip-top-level-dir = True
    md5sum = d3d25e6442e14dafb3d45f44094fd377
    destination = lib

    [versions]
    MarkupSafe = 0.15
    Pillow = 1.7.7
    PyXML = 0.8.4
    babel = 0.9.6
    feedparser = 5.1.1
    gdata = 2.0.16
    lxml = 2.3.3
    mako = 0.6.2
    psycopg2 = 2.4.4
    pychart = 1.39
    pydot = 1.0.28
    pyparsing = 1.5.6
    python-dateutil = 1.5
    python-ldap = 2.4.10
    python-openid = 2.2.5
    pytz = 2012b
    pywebdav = 0.9.4.1
    pyyaml = 3.10
    reportlab = 2.5
    simplejson = 2.4.0
    vatnumber = 1.0
    vobject = 0.8.1c
    werkzeug = 0.8.3
    xlwt = 0.7.3
    zc.recipe.egg = 2.0.0
    zsi = 2.0-rc3

.. warning:: aeroo requires Genshi 0.6 and doesn't work with Genshi 0.7 because of the unicode encoding problems.

copy configuration file to the project directory and start buildout::

    (OpenERP_6.1)$ buildout bootstrap
    (OpenERP_6.1)$ buildout

When buildout is finished you have new OpenERP installation.
You can start server with this command::

    (openerp_61_ocb)$ bin/start_openerp

Build out place OpenERP
Edit etc/epenerp.cfg (senza spazio dopo la virgola):
In particular in this configuration file you should control::

    options.admin_passwd = admin
    db_user = openerp
    db_password = xxxxxx
    log_handler = [':DEBUG']
    

From this moment on, we can start OpenERP server this way.
First activate Virtual Environment::

    $ workon openerp_61_ocb

And then start OpenERP server (and client)::

    (openerp_61_ocb)$ bin/start_openerp
    

GTK Client
==========    

.. note:: it is difficult to install GTK Client inside virtualenv, because it is hard to compile gtk2 inside virtualenv.

Download the latest client from http://nightly.openerp.com/6.1/releases/


Reinstall of VirtualEnv after upgrade from Ubuntu 12.04 LTS to 14.04 LTS
======================================================================

After upgrade of the distribution I upgraded **virtualenv** and **pip**. After that it was required to recreate Virtual Env::

    $ mkvirtualenv openerp_60

After creating new environment I installed required python libraries:

Required by Server (installed with PIP)::

    lxml
    psycopg2
    PyYAML
    mako
    python-dateutil
    pydot
    reportlab
    vobject
    xlwt
    xlrd


Required by Client (installed with PIP)::

    cherrypy
    formencode==1.2.4
    simplejson
    babel

I'm trying to use PIP everywhere it is possible, but when I can't I use APT-GET.

Required by Server (installed with  APT-GET)::

    python-egenix-mxdatetime 
    python-pychart
    python-tz
    
    
.. note:: A good tutorial for installation of OpenERP 6.0 http://wiki.odoo-italia.org/doku.php/area_tecnica/installazione/v6_ubuntu_10.04



