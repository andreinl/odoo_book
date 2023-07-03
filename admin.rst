Server System Administration
****************************

Solving database problems
=========================

Corrupted Database
------------------

Reconstruct database from command line (update module base)::

    $ ./openerp-server -c ../etc/openerp.cfg  -d Trepi_1512 -u base


Databse base is broken because new version of modules requires not installed module.

Install module from command line::

    $ ./openerp-server -c ../etc/openerp.cfg  -d Trepi_1512 -i base_action_rule_triggers


Database backup e restore
-------------------------

Sometime direct link to this URLs can be hidden, so you should know where to find them.
In v.8::

    http://localhost:8069/web/database/selector
    http://localhost:8069/web/database/manager


Database backup e restore from command line
-------------------------------------------

Create database copy from command line::

    $ pg_dump Vampistore_2704 -U openerp -W > Vampigroup_2704.dump

Restore database copy from dump::

    $ createdb -T template0 Vampistore_2704 -U openerp -W
    $ psql Vampistore_2704 < Vampigroup_2704.dump -U openerp -W

if combined format was used::

    $ pg_restore -d Trepi_0605 -U openerp -W --role=openerp trepi.sql
