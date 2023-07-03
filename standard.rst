Style Guide for OpenERP Code
****************************

This document describes programming conventions that should be used when writing modules for OpenERP. First of all this conventions include variable names. They are not required for python code to be executed, but using this conventions make your code more user friendly and more understandable by other programmers. This conventions don't substitute PEP8, so any time you finished programming something make a check with flake8 (https://pypi.python.org/pypi/flake8)


Object representation
=====================

This is right::

    product_model = self.pool.get('product.product')

From version 6.1 of OpenERP, this can be written as::

    product_model = self.pool['product.product']

From version 8.0 of Odoo::

    product_model = self.env['product.product']

This is wrong::

    product = self.pool.get('product.product')
    

many2one field
==============

This is right::

    _columns = {
        'product_id': fields.many2one('product.product', _('Product'))
    }

From version 8.0 of Odoo::

    product_id = fields.Many2one('product.product', _('Product'))

This is wrong::

    _columns = {
        'product': fields.many2one('product.product', _('Product'))
    }


one2many field
==============

This is right::

    _columns = {
        'expense_ids': fields.one2many('hr.expense.line', 'task_id', _('Expenses')),
    }

From version 8.0 of Odoo::

    expense_ids = fields.One2many('hr.expense.line', 'task_id', _('Expenses'))

This is wrong::
    
    _columns = {
        'expenses': fields.one2many('hr.expense.line', 'task_id', 'Expenses'),
    }


Object definition
=================

This is right::

    product = product_model.browse(cr, uid, product_id, context=context)

From version 8.0 of Odoo::

    product = product_model.browse(product_id)

This is wrong::

    product_product_model = product_model.browse(cr, uid, product_id)
    
.. note:: It is important always include context=context as this is needed for some internal OpenERP processes (like translations).
    

Raw SQL
=======

Never ever use raw SQL. Write database independent modules. If you absolutely need to use raw SQL remember that keywords should be capitalized::

This is right::

    cr.execute("""
        SELECT a.id
          FROM hr_attendance a
         INNER JOIN hr_employee e
               INNER JOIN resource_resource r
                       ON (e.resource_id = r.id)
            ON (a.employee_id = e.id)
        WHERE %(date_to)s >= date_trunc('day', a.name)
              AND %(date_from)s <= a.name
              AND %(user_id)s = r.user_id
         GROUP BY a.id""", {'date_from': ts.date_from,
                            'date_to': ts.date_to,
                            'user_id': ts.employee_id.user_id.id})

This is wrong::

    cr.execute("""create or replace view hr_timesheet_sheet_sheet_account as (
        select
            min(hrt.id) as id,
            l.account_id as name,
            s.id as sheet_id,
            sum(l.unit_amount) as total,
            l.to_invoice as invoice_rate
        from
            hr_analytic_timesheet hrt
            left join (account_analytic_line l
                LEFT JOIN hr_timesheet_sheet_sheet s
                    ON (s.date_to >= l.date
                        AND s.date_from <= l.date
                        AND s.user_id = l.user_id))
                on (l.id = hrt.line_id)
        group by l.account_id, s.id, l.to_invoice
    )""")


Python reserved words
=====================

Never use reserved words as variable names.

This is wrong::

    for id in ids:
        print id

**id()** is a built-in function that gives the memory address of an object.


Function Names
==============

Function names should be lowercase, with words separated by underscores as necessary to improve readability.

This is right::

    def my_new_function(values):
        pass
    

This is wrong::

    def myNewFunction(values):
        pass


Method Names and Instance Variables
===================================

Use the function naming rules: lowercase with words separated by underscores as necessary to improve readability.

Use one leading underscore only for non-public methods and instance variables.

This is right::

    def my_new_function(self, cr, uid, ids, value):
        pass


This is wrong::

    def myNewFunction(self, cr, uid, ids, value):
        pass


Standard OpenERP variable names
===============================

- **uid** - user Id
- **cr** - cursor
- **ids** or **id** - id or ids of an object we are working with.  


Class Names
===========

In Python class names should normally use the CapWords convention. This goes against official OpenERP style, but is widely adopted by OCA programmers.


Searching criteria
==================

The right name for a variable is **domain**::

    domain = [('state', '=', 'valid'), ('check_support', '=', True)]


Imports
=======

.. note:: Starting from the OpenERP 6.1 **osv** is depricated.

Use the full openerp namespace, do not use::

    import osv, fields
    from tools.translate import _

But::

    from openerp.osv import orm, fields
    from openerp.tools.translate import _
    
When importing Python modules from the same addon, use explicit relative import rather than absolute import, example in __init__.py, do not use::

    import sale

But::

    from . import sale


Models
======

.. note:: Starting from the OpenERP 6.1 **osv** is deprecated.

Do not use::

    class my_class(osv.osv):
        pass
        
    class my_transient_class(osv.osv_memory):
        pass

From version 6.1 of OpenERP::
    
    class MyClass(orm.Model):
        pass
        
    class my_transient_class(orm.TransientModel):
        pass

From version 8.0 of Odoo::

    class MyClass(models.Model)

Do not instantiate the model classes after their declaration

When you create a new model, do not forget the `_description` attribute.


Time format 
===========

Do not write  your own time formatter ("%Y-%m-%d"), but use Openerp constants::
  
    from openerp.tools import DEFAULT_SERVER_DATE_FORMAT 
    
or (for datetime):: 

    from openerp.tools import DEFAULT_SERVER_DATETIME_FORMAT 
     
and use it in this way::

    date_variable.strftime(DEFAULT_SERVER_DATE_FORMAT)
    datetime.strptime(date_variable, DEFAULT_SERVER_DATE_FORMAT)
    
or::

    datetime_variable.strftime(DEFAULT_SERVER_DATETIME_FORMAT) 
    datetime.strptime(datetime_variable, DEFAULT_SERVER_DATETIME_FORMAT)


Translations
============   
    
There are some messages written in code that must be translatable (rewrite of fields labels, user messages, etc).

To do it, make this import (version 6.1 of OpenERP)::
    
    from openerp.tools.translate import _

From version 8.0 of Odoo::

    from openerp import _

See this link for more tips about how to correctly handle this translatable texts:
https://doc.openerp.com/contribute/15_guidelines/coding_guidelines_framework/#use-the-gettext-method-correctly
   

Exception Management
====================
   
 - Do not use  **osv.except_osv**, it is deprecated.
 - Use **orm.except_orm** when error is not technical and should be useful to end user (and maybe translated)::

    raise orm.except_orm(_('Error'), _('No Asset selected, nothing to reserve'))

   From version 8.0 of Odoo **except_orm** exception is deprecated. We should use openerp.exceptions (Ex.: Warning) and subclasses instances::

    from openerp import exceptions

    raise exceptions.ValidationError(_("The beginning and the end of itinerary can't be the same"))

 - Try to use Python Builtin exception when appropriate see: http://docs.python.org/2/library/exceptions.html
 - Do not translate technical exception messages.
 - Avoid exclamation mark (!) in error messages


Context
=======

It can happen that context is not passed to the function and it's value is *None*. We can get a lot information just from the user id, so instead of assigning to context an empty dictionary use this variant::


    context = context or self.pool['res.users'].context_get(cr, uid)

To not make the query when context dictionary is already {} (maybe you can be interested in not propagating user context and you pass explicitly context={} in your call to the function).


Acknowledgments
---------------

Thanks to Pedro Manuel Baeza Romero for useful suggestions.

