Appendix A
**********

Upgrades in OpenERP 7.0 as compared to 6.1
============================================

Removed elements:
-----------------

.. highlight:: xml

* Seems that 7.0 don't like strings like this::

    <label colspan="6" width="220"/>


* Setting the `type` field is deprecated in the `ir.ui.view` model



Upgrades in OpenERP 6.1 as compared to 6.0.x
============================================

Here are some of the changes to be kept in mind while switching to this improved and smarter version. This is for developers switching to 6.1 version from 6.0.x-

View:
-----

* The web is integrated in server, so we just have to start the server and web automatically starts at http://localhost:8069 by default.
* By defaut the session of web does not time out just like in GTK client
* Unlike in 6.0.x, one2many field does not save in database even if Save & New or Save & Close button is clicked. To actually Save it in database, one have to click on main Save button of parent object.
* If a menu is right clicked to open in new tab or window it does not open that menu. Rather it takes you to the home screen.
* Remembers the last database accessed. This removes the pain of selecting the right database each time you log in, specially when you have multiple test databases.

.. highlight:: xml

* New functionality::

    <field name="state" widget="statusbar" statusbar_visible="draft,confirmed,done" statusbar_colors='{"exception":"red","cancel":"red"}'/>




Coding:
-------

* Now, one menu cannot be clicked to open an action and also to open its child menus. If a menu has child menus, then it cannot have action assigned to it.
* many2many declaration is simplified. The relation name and relation columns need not be specified explicitly. Only this is enough- 'invoice_ids': fields.many2many('account.invoice',string='Invoices')
* For circular reference, there is no need to split classes in two. (Example taken from OpenERP 6.1 Release Notes) 

.. highlight:: python

In v6.0, we used to write the following code::

    class res_groups(osv.osv):
        _name = 'res.groups'
        _columns = {
            'name': fields.char('Name'),
        }
    
    res_groups()


    class res_users(osv.osv):
        _name = 'res.users'
        _columns = {
            'name': fields.char('Name'),
            'group_ids': fields.many2many('res.users', string='Users'),
        }

    res_users()


    class res_groups2(osv.osv):
        _inherit= 'res.groups'
        _columns = {
            'user_ids': fields.many2many('res.users', string='Users'),
        }

    res_groups2()


As of 6.1, this code can be reduced to::

    class res_groups(osv.osv):
        _name = 'res.groups'
        _columns = {
            'name': fields.char('Name'),
            'user_ids': fields.many2many('res.users', string='Users'),
        }

    class res_users(osv.osv):
        _name = 'res.users'
        _columns = {
            'name': fields.char('Name'),
            'group_ids': fields.many2many('res.users', string='Users'),
        }

* The debug mode can be turned on by simply appending *?debug* to the OpenERP URL.
* *netsvc.Logger()* is depricated. Standard Python *logging* module should be used instead.
* As of 6.1 datetime field is saved in UTC format. To retrive data in clients format function context_timestamp should be used::

    from openerp.osv import fields
    from datetime import datetime
    my_date = fields.datetime.context_timestamp(cr, uid, datetime.now(), context=context)


