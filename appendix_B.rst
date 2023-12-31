Appendix B. openerp-server.conf for OpenERP 7 explained
*******************************************************

by **Nikola Stojanoski** (http://www.vionblog.com/openerp-server-conf-for-openerp-7-explained/)

Here are the options that you can use in your openerp-server.conf file to tweak your OpenERP 7 installation.

You can start your server with a specified config file with -c command.::

    ./server/openerp-server -c /path/to/openerp-server.conf


Here is the config file spitted  into parts for easy understanding.

Server startup config – Common options
======================================
::

    # Admin password for creating, restoring and backing up databases
    admin_passwd = admin

    # default CSV separator for import and export
    csv_internal_sep = ,

    # to compress reports
    reportgz = False

    # disable loading demo data for modules to be installed (comma-separated, use "all" for all modules)
    without_demo = False

    # Use this for big data importation, if it crashes you will be able to continue at the current state. Provide a filename to store intermediate importation states.
    import_partial = 

    # file where the server pid will be stored
    pidfile = None

    # specify additional addons paths (separated by commas)
    addons_path = /full/path/to/addons
    
    # Comma-separated list of server-wide modules default=web
    server_wide_modules = None
    
    
XML-RPC / HTTP – XML-RPC Configuration
======================================
::

    # disable the XML-RPC protocol
    xmlrpc = True

    # Specify the TCP IP address for the XML-RPC protocol. The empty string binds to all interfaces.
    xmlrpc_interface = 

    # specify the TCP port for the XML-RPC protocol
    xmlrpc_port = 8069

    # Enable correct behavior when behind a reverse proxy
    proxy_mode = False


XML-RPC / HTTPS – XML-RPC Secure Configuration
==============================================
::

    # disable the XML-RPC Secure protocol
    xmlrpcs = True

    # Specify the TCP IP address for the XML-RPC Secure protocol. The empty string binds to all interfaces.
    xmlrpcs_interface = 

    # specify the TCP port for the XML-RPC Secure protocol
    xmlrpcs_port = 8071

    # specify the certificate file for the SSL connection
    secure_cert_file = server.cert

    # specify the private key file for the SSL connection
    secure_pkey_file = server.pkey

NET-RPC – NET-RPC Configuration
===============================
::

    # enable the NETRPC protocol
    netrpc = False

    # specify the TCP IP address for the NETRPC protocol
    netrpc_interface = 

    # specify the TCP port for the NETRPC protocol
    netrpc_port = 8070

WEB – Web interface Configuration
=================================
::

    # Filter listed database REGEXP
    dbfilter = .*


Static HTTP – Static HTTP service
=================================
::

    # enable static HTTP service for serving plain HTML files
    static_http_enable = False 

    # specify the directory containing your static HTML files (e.g '/var/www/')
    static_http_document_root = None

    # specify the URL root prefix where you want web browsers to access your static HTML files (e.g '/')
    static_http_url_prefix = None


Testing Group – Testing Configuration
=====================================
::

    # Launch a YML test file.
    test_file = False

    # If set, will save sample of all reports in this directory.
    test_report_directory = False

    # Enable YAML and unit tests.
    test_enable = False

    # Commit database changes performed by YAML or XML tests.
    test_commit = False

Logging Group – Logging Configuration
=====================================
::

    # file where the server log will be stored
    logfile = None

    # do not rotate the logfile
    logrotate = True

    # Send the log to the syslog server
    syslog = False

    # setup a handler at LEVEL for a given PREFIX. An empty PREFIX indicates the root logger. This option can be repeated. Example: "openerp.orm:DEBUG" or "werkzeug:CRITICAL" (default: ":INFO")
    log_handler = [':INFO']

    # specify the level of the logging. Accepted values: info, debug_rpc, warn, test, critical, debug_sql, error, debug, debug_rpc_answer, notset
    log_level = info


SMTP Group – SMTP Configuration
===============================
::

    # specify the SMTP email address for sending email
    email_from = False 

    # specify the SMTP server for sending email
    smtp_server = localhost 

    # specify the SMTP port
    smtp_port = 25 

    # specify the SMTP server support SSL or not
    smtp_ssl = False 

    # specify the SMTP username for sending email
    smtp_user = False

    # specify the SMTP password for sending email
    smtp_password = False


Database related options
========================
::

    # specify the database name
    db_name = False

    # specify the database user name
    db_user = openerp

    # specify the database password
    db_password = False

    # specify the pg executable path
    pg_path = None

    # specify the database host
    db_host = False

    # specify the database port
    db_port = False

    # specify the the maximum number of physical connections to posgresql
    db_maxconn = 64

    # specify a custom database template to create a new database
    db_template = template1


Internationalisation options
============================
::

    translate_modules = ['all']


Security-related options
========================
::

    # disable the ability to return the list of databases
    list_db = True
    Advanced options – Advanced options

    # enable debug mode
    debug_mode = False

    # specify reference timezone for the server (e.g. Europe/Brussels")
    timezone = False

    # Force a limit on the maximum number of records kept in the virtual osv_memory tables. The default is False, which means no count-based limit. 
    osv_memory_count_limit = False 

    # Force a limit on the maximum age of records kept in the virtual osv_memory tables. This is a decimal value expressed in hours, and the default is 1 hour.
    osv_memory_age_limit = 1.0 

    # Maximum number of threads processing concurrently cron jobs (default 2)
    max_cron_threads = 2

    # Use the unaccent function provided by the database when available.
    unaccent = False


Multiprocessing options
=======================
::

    # Specify the number of workers, 0 disable prefork mode.
    workers = 0

    # Maximum allowed virtual memory per worker, when reached the worker be reset after the current request (default 671088640 aka 640MB)
    limit_memory_soft = 671088640

    # Maximum allowed virtual memory per worker, when reached, any memory allocation will fail (default 805306368 aka 768MB)
    limit_memory_hard = 805306368

    # Maximum allowed CPU time per request (default 60)
    limit_time_cpu = 60

    # Maximum allowed Real time per request (default 120)
    limit_time_real = 120

    # Maximum number of request to be processed per worker (default 8192)
    limit_request = 8192


There are few more options that you can find in this file::

    vi server/openerp/tools/config.py


