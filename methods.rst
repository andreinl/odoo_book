ORM Methods of orm.Model or models.Model (new API) objects
************************************************

_description
============

A short description of a class

.. note:: A description is saved in string char(64). Take it under control. It's easy to exceed this limit.


create(self, cr, uid, values, context=None)
===========================================

Returns id of the new record

Sometime you need to create a record that has a subrecords, in this case you have 2 possibilitie. You can create subrecords first and than create a record::

    rent_line_ids = []
    for rentable in rentables:
        rent_line_ids.append(self.pool['select.rent.lines'].create(cr, uid, {'name': rentable.name}))
    
    select_asset_id = self.pool['select.rent.asset'].create(cr, uid, {
        'name': order.name,
        'rent_line_ids': [(6, 0, rent_line_ids)],
    })


Or you can pass a list of dictionaries with the values of subrecords::

    rent_lines = []
    for rentable in rentables:
        rent_lines.append({'name': rentable.name})
    
    select_asset_id = self.pool['select.rent.asset'].create(cr, uid, {
        'name': order.name,
        'rent_line_ids': [(0, 0, line) for line in rent_lines],
    })


.. note::
 * (0, False, { values })    link to a new record that needs to be created with the given values dictionary
 * (1, ID, { values })    update the linked record with id = ID (write *values* on it)
 * (2, ID)                remove and delete the linked record with id = ID (calls unlink on ID, that will delete  the object completely, and the link to it as well)
 * (3, ID)                cut the link to the linked record with id = ID (delete the relationship between the two objects but does not delete the target object itself)
 * (4, ID)                link to existing record with id = ID (adds a relationship)
 * \(5\)                    unlink all (like using (3,ID) for all linked records)
 * (6, False, [IDs])          replace the list of linked IDs (like using (5) then (4, ID) for each ID in the list of IDs)


write(self, cr, uid, ids, values, context=None)
===============================================

.. note:: ids can be integer, long or list. We should handle it when writing our custom write() method.

::

    def write(self, cr, uid, ids, values, context=None):
        if not ids:
            return False
        if isinstance(ids, (int, long)):
            ids = [ids]
        
        openerp_result = super(hr_contractor, self).write(cr, uid, ids, values, context)


copy(self, cr, uid, order_id, defaults, context=None)
=====================================================

This function is called when we try to duplicate an object. Unfortunately it will also dupplicate fields that contain sequence related fields. In this case we should delete this values and remove fields from values passed to *create* function::

    def create(self, cr, uid, values, context=None):
        if 'name' in values and not values['name']:
            del values['name']
        return super(repair_order, self).create(cr, uid, values, context)
    
    def copy(self, cr, uid, order_id, defaults, context=None):
        defaults['name'] = None
        return super(repair_order, self).copy(cr, uid, order_id, defaults, context)


copy_data(self, cr, uid, order_id, default=None, context=None)
==============================================================

When you copy a model that contains any relational fields this method is called on linked fields.


unlink(self, cr, uid, ids, context=None)
========================================

.. note:: ids should be a list

onchange_field_name(self, cr, uid, ids, variable, context=None)
=================================================================

.. note:: This is not a standard field but a convention used in OpenERP

Returning dictionary has 3 standard fields: {'value': result, 'domain': domain, 'warning': warning}

We should call this method from XML of the field which changes we need to handle:

.. code-block:: guess

    <field name="new_prodlot_code" on_change="onchange_new_prodlot_code(new_prodlot_code, product_id, prodlot_id)" />


Example::
    
    def onchange_foglio_fine(self, cr, uid, ids, foglio_fine, context=None):
        val = False
        if foglio_fine:
            val = time.strftime('%Y-%m-%d')
        return {'value': {'dt_foglio_fine': val}}

.. note:: instead of *raise* inside onchange_smth() function we should use *warning* key in the returning dictionary 

::

     return {'value': {}, 'warning': {'title': _('Warning!'), 'message': _('Non esiste Tipo di Installazione per il prodotto "{product}"'.format(product=order_line.product_id.name))}}


An example of changing *domain* dynamically::

    _columns = {
        'serial_number': fields.many2one('stock.production.lot', "Serial Number", ondelete="no action", required=False),
    }
    
    def _get_assets_serials(self, cr, uid, ids, product_id, context=None):
        stock_move_obj = self.pool.get('stock.move')
        serials = []
        
        asset_location_ids = self.pool.get('stock.location').search(cr, uid, [('usage', '=', 'assets')])
        for location_id in asset_location_ids:
            stock_move_ids = stock_move_obj.search(cr, uid, [('location_dest_id', '=', location_id)])
            
            ### TODO: order_by and only take serials that are still in location          
            stock_move = stock_move_obj.browse(cr, uid, stock_move_ids)
            [serials.append(row.prodlot_id.name) for row in stock_move if not str(row.prodlot_id.name) == 'None']
        
        return serials
   
    def onchange_product_id(self, cr, uid, ids, product_id, context=None):
        # TODO: We should show only non assigned serial numbers if product split type is Single
        has_date_option = False
        res_partner = ''
        serial_number = []
        product_product_id = 0
        if product_id:
            product_id_obj = self.pool.get('asset.product')
            products = product_id_obj.browse(cr, uid, [product_id], context)
            if products and products[0].has_date_option == True:
                has_date_option = True
            res_partner = products[0].manufacturer.name
            product_product_id = products[0].product_product_id.id
        
        assets_serials = self._get_assets_serials(cr, uid, ids, product_id, context)

        return {
            'value': {'has_date_option': has_date_option, 'res_partner': res_partner,},
            'domain': {'serial_number': [('product_id', '=', product_product_id), ('name', 'in', assets_serials)]},
        }

Sometimes you need to change values inside many2one related lines. The problem is that you should sent to view all values, not only the one that is changed. There is a function which is called resolve_o2m_commands_to_record_dicts() (From v.7: resolve_2many_commands) that should help you to set values. Unfortunately seems that this function don't like reference fields and even worse it produces a dictionary that reppresent many2one fields as two members lists which is not suitable for write() function.

This is an example of how to deal with this problems::

    def onchange_date(self, cr, uid, ids, date, context=None):
        stock_move_obj = self.pool['stock.move']
        
        new_fields_to_read = []
        
        for key in stock_move_obj._columns.keys():
            reserved_keys = ('create_date', 'write_date', 'create_uid', 'write_uid', 'company_id', 'id')
            # We should exclude reference fields, because resolve_o2m_commands_to_record_dicts don't know to handle them
            if not key in reserved_keys and not stock_move_obj._columns[key]._type in ('reference', ):
                new_fields_to_read.append(key)
        
        o2m_commands = []
        
        stock_picking = self.browse(cr, uid, ids[0], context)
        for stock_move in stock_picking.move_lines:
            o2m_commands.append([1, stock_move.id, {'date': date}])
        
        values = self.resolve_o2m_commands_to_record_dicts(cr, uid, 'move_lines', o2m_commands, new_fields_to_read, context=context)
        
        for line in values:
            for key in line.keys():
                # Here we are looking for many2one fields, but we should exclude functions which returns integers instead of lists,
                # at the same time we should include related fields which are functions, but return lists
                if not key in reserved_keys and stock_move_obj._columns[key]._type == 'many2one' and (not hasattr(stock_move_obj._columns[key], '_fnct') or hasattr(stock_move_obj._columns[key], 'relation')):
                    # we should write related field id instead of a list:
                    line[key] = line[key] and line[key][0] or False
        
        return {'value': {'move_lines': values}}
        
.. note:: However there is another problem with changing values inside many2one lines: all changes inside many2one lines that were made from view will be lost, because resolve_o2m_commands_to_record_dicts() takes values from database.

Return a Warning message
------------------------
to return a warning with the **new api**::

    @api.one
    @api.onchange('partner_id')
    def onchange_partner_id(self):
        return {'warning': {'title': 'Warning', 'message': 'Message'}}

.. note::  A module should be updated to make this work even if it's only Python code.


search(self, cr, uid, args, offset=0, limit=0, order=None, context=None, count=False)
=====================================================================================

This function is called when search button is pressed.

If like or ilike is used in args - this is string
complete_name - field name on which we want to apply our improvement
name - this means that we make a search on a function field complete_name and to have a result we need to use a real field

.. note:: OpenERP automatically add % at the and in the beginning of the string, so if we are looking for a 'product' it became '%product%'. In SQL this means that we are looking for a 'product' in any position inside a string, so 'Big product' or 'production' will match. If we want exact matching we should use '=like' instead of 'like' and '=ilike' instead of 'ilike' (for case insensitive search).

If we want to give possibility to search for more than one word, we can achieve it this way::

    def search(self, cr, uid, args, offset=0, limit=0, order=None, context=None, count=False):
        """
            ilike - this way we know, that it is a string
            partner_id - field name on which we want to apply our improvement
        """
        new_args = []
        
        for arg in args:
            if arg and len(arg)==3 and arg[0] in ('partner_id', ) and arg[1]=='ilike':
                arg = ('partner_id', 'ilike', arg[2].replace(' ', '%'))
                new_args.append(arg)
            else:
                new_args.append(arg)
               
        return super(project_protocol, self).search(cr, uid, new_args, offset=offset, limit=limit, order=order, context=context, count=count)
 

To make a search on many2one field  we should write a function like this::

    _columns = {
        'letter_id': fields.many2one('res.letter', 'Protocol', required=False),
    }    
    
    def search(self, cr, uid, args, offset=0, limit=0, order=None, context=None, count=False):
        new_args = []
        for arg in args:
            if len(arg) == 3 and arg[0] == 'letter_id':
                letter_ids = self.pool.get('res.letter').search(cr, uid, [('number', 'ilike', arg[2])])
                new_args.append(('letter_id', 'in', letter_ids))
            else:
                new_args.append(arg)
                
        return super(sale_order, self).search(cr, uid, new_args, offset=offset, limit=limit, order=order, context=context, count=count)

A valid **order** specification is a comma-separated list of valid field names (optionally followed by **asc/desc** for the direction)

For example::

    first_invoice_id = self.search(cr, uid, [('date_invoice', '!=', False)], order='date_invoice asc', limit=1)

Complex domain example::

    domain = [
        '&', '|', '&',
        ('date_start', '<=', date_start), ('date_end', '>=', date_start),
        '&',
        ('date_end', '>=', date_start), ('date_end', '<=', date_end),
        ('asset_id', '=', asset_id)
    ]

This expression will be transformed in this query::

    ((((asset_rent_period."date_start" <= %s)  AND  (asset_rent_period."date_end" >= %s))  OR  ((asset_rent_period."date_end" >= %s)  AND  (asset_rent_period."date_end" <= %s)))  AND  (asset_rent_period."asset_id" = %s))

.. highlight:: xml

Example of extending search field of a product by substituting default domain (New API)
---------------------------------------------------------------------------------------
In this example we add a fake column "part_number"::

    <record id="product_template_part_number_search_view" model="ir.ui.view">
        <field name="name">product.template.part.number.search</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_search_view"/>
        <field name="arch" type="xml">
            <data>
                <field name="name" position="attributes">
                    <attribute name="filter_domain">['|', '|', ('default_code', 'ilike', self), ('name', 'ilike', self), ('part_number', 'ilike', self)]</attribute>
                </field>
            </data>
        </field>
    </record>

.. highlight:: python

In the **search** function we intercept an argument that contains our fake expression and substitute it with the good one::

    @api.model
    def search(self, args, offset=0, limit=0, order=None, count=False):
        for i, arg in enumerate(args):
            print arg
            if len(arg) > 2 and (arg[1] == 'like' or arg[1] == 'ilike') and arg[0] == 'part_number':
                part_numbers = self.env['spare.part.number'].search([('name', 'ilike', arg[2])])
                if part_numbers:
                    args[i] = ['part_number_id', 'in', [part_number.id for part_number in part_numbers]]

        return super(ProductTemplate, self).search(args, offset=offset, limit=limit, order=order, count=count)


name_search(self, cr, user, name, args=None, operator='ilike', context=None, limit=100)
=======================================================================================

This function is called during digitalization in search field on class which has many2one relationship with this table::

    def name_search(self, cr, uid, name, args=None, operator='ilike', context=None, limit=100):
        if args and len(args[0])==3 and args[0][1]=='ilike':
            ## Ex: args = [('name', 'ilike', 'Q24M%nero')]
            args = [(args[0][0], 'ilike', args[0][2].replace(' ', '%'))]
        elif operator == 'ilike' and name:
            name = name.replace(' ', '%')
        return super(project_place, self).name_search(cr, uid, name, args, operator, context, limit)


Function returns a list which is created when we pass line_ids to name_get() function. In this example we add a search on field 'number' to the search on field 'name'::

    def name_search(self, cr, uid, name, args=None, operator='ilike', context=None, limit=100):
        res = super(res_letter, self).name_search(cr, uid, name, args, operator, context, limit)
        letter_ids = self.search(cr, uid, [('number', 'ilike', name)])
        return res + self.name_get(cr, uid, letter_ids)


.. note:: List res + self.name_get(cr, uid, letter_ids) can contain duplicate values. To solve this problem we should write it this way: list(set(res + self.name_get(cr, uid, letter_ids)))


Sometimes we need a result of the search inside related table::

    def name_search(self, cr, uid, name, args=None, operator='ilike', context=None, limit=100):
        if operator == 'ilike' and name:
            name = name.replace(' ', '%')
            #product_name = self.pool.get('asset.product').name_search(cr, user, name, args, operator, context, limit)
            query = """SELECT asset.id, asset.track_no, product.name_template
                        FROM (asset_asset AS asset 
                            LEFT JOIN asset_product AS a_product
                            ON asset.product_id = a_product.id)
                        LEFT JOIN product_product AS product
                            ON a_product.product_product_id = product.id
                        WHERE product.name_template ILIKE '%{0}%'""".format(name)
            cr.execute(query)
            assets = cr.fetchall()
            res = [(asset[0], '[' + asset[1] + '] ' + asset[2]) for asset in assets]
            return res
            
        elif args and len(args[0])==3 and args[0][1]=='ilike':
            ## Will never happen (?)
            ## Ex: args = [('name', 'ilike', 'Q24M%nero')]
            args = [(args[0][0], 'ilike', args[0][2].replace(' ', '%'))]
            
        return super(asset_asset, self).name_search(cr, uid, name, args, operator, context, limit)


.. note:: Seems to be an old example, may be we can do it better

::

    def name_search(self, cr, uid, name, args=None, operator='ilike', context=None, limit=100):
        ## This is the right way, but requires rewriting of the search function: 
        #sim_ids = self.search(cr, uid, ['|', '|', ('sim_internal_number', 'ilike', name), ('prefix_number', 'ilike', name), ('number', 'ilike', name)])
        sim_ids = self.search(cr, uid, [('sim_internal_number', 'ilike', name + '%')])
        sims = self.browse(cr, uid, sim_ids)
        res = [(sim.id, '[' + sim.sim_internal_number + '] ' + sim.prefix_number + ' ' + sim.number) for sim in sims]
        return res 


Sorted results
----------------
::

    def name_search(self, cr, uid, name, args=None, operator='ilike', context=None, limit=100):
        task_selection = super(project_task, self).name_search(cr, uid, name, args, operator, context=context, limit=limit)
        # Sort by name
        return sorted(task_selection, key=lambda x: x[1])


name_get(self, cr, uid, ids, context=None)
==========================================

The name that will be shown inside tree::

    def name_get(self, cr, uid, ids, context=None):
        if not len(ids):
            return []
        res = []
        if not context:
            context = self.pool['res.users'].context_get(cr, uid)
        length = context.get('name_lenght', False) or 80
        for record in self.browse(cr, uid, ids, context=context):
            name = record.complete_name or record.name or ''
            res.append((record.id, name))
        return res


address_get(self, cr, uid, ids, adr_pref=None)
==============================================

.. note:: valid only for table res_partner


adr_pref - address type. Default is ['default']


perm_read(cr, user, ids, context=None, details=True)
====================================================

If you need to get access to system fields of a record, this function should be used.

For example to read write_date::

    perms = self.perm_read(cr, uid, ids)
    write_date = perms[0].get('write_date', False)

.. note:: ids should be a list


default_get(self, cr, uid, fields, context=None)
================================================

Function returns a dictionary with default values::

    @api.model
    def default_get(self, fields):
        order_ids = self.env['broker.purchase.order'].browse(self._context['active_id']).order_ids.ids
        values = super(WizardDistributionList, self).default_get(fields)

        self._cr.execute("""SELECT truck_info_id FROM sale_order_line
            WHERE order_id in ({orders})
            GROUP BY truck_info_id
        """.format(orders=', '.join([str(order_id) for order_id in order_ids])))

        values['truck_info_ids'] = [truck_info[0] for truck_info in self._cr.fetchall()]
        return values


get_object_reference(self, cr, uid, module, xml_id)
===================================================

This function is used to get an id (res_id in 'ir.model.data') of a view ('ir.ui.view')::
    
    form_res = self.pool.get('ir.model.data').get_object_reference(cr, uid, 'sale_rent', 'select_rent_asset')
    form_id = form_res and form_res[1] or False


browse(self, cr, uid, object_id, context)
=========================================

**by Fabien Pinckaers**

Using read() is a bad practice. read() is used for web-services calls
but in your own method calls you should always use browse(). Not only,
it allows s a better quality of the code, but it's also better for the
performance.

    - **read()** calls name_get for many2one fields producing extra SQL queries you probably don't need.
    - **browse()** is optimized for prefetching and auto-load of fields.

It's true that browse() may load a few fields you do not need (not all).
It prefetches stored fields, because those fields do not costs anything
to load in terms of performance.

It's very complex to optimize for performance with read() when the code
is complex (involves loops, other method calls). Whereas, with browse(),
the framework do the optimization job for you.

Usually, code implemented with read are often with complexities of O(n)
or O(n²) as soon as there is loops in your methods. But codes written
with browse() are automatically O(1) if browse is called at the
beginning of your computations. (not inside loops)

Want a small example? Try this code on res.partner::

    for company in self.browse(cr, uid, ids, context=context):
        for people in company.child_ids:
            print company.name, people.country_id.code

The above will do 6 SQL queries, whatever the length of ids and number
of people/countries. (if IDS>200, he will split into subqueries)

The same code with read(), you will probably end up with 3*len(ids) + 3
queries.


Long story short: browse() scale, read() not. (even in v7 or preceding
versions)


Standard methods
================

::

    'read',
    'write',
    'create',
    'default_get',
    'perm_read',
    'unlink',
    'fields_get',
    'fields_view_get',
    'search',
    'name_get',
    'distinct_field_get',
    'name_search',
    'copy',
    'import_data',
    'search_count',
    'exists'


Domain filters
==============

OpenERP uses Polish Notation for Domain filters.
First you should understand what is polish notation. You can find detailed information in wikipedia about polish notation. http://en.wikipedia.org/wiki/Polish_notation

Example:
( A OR B ) AND ( C OR D OR E )
should be converted to the polish notation as

  AND OR A B OR OR C D E

And should be solved by the algorithm with following order [] represents operation

  AND [OR A B] OR OR C D E         Result of [OR A B] is F

  AND F OR [OR C D] E              Result of [OR C D] is G

  AND F [OR G E]                   Result of [OR G E] is H

  [AND F H]

it starts from LEFT to Right.

"If another operator is found before two operands are found, then the old operator is placed aside until this new operator is resolved. This process iterates until an operator is resolved, which must happen eventually, as there must be one more operand than there are operators in a complete statement." From wikipedia article.


Creating table indexes
======================

Indexes are very important for performance. OpenERP has no standard methods for index creation. _auto_init() method can be used for this purpose::


    _index_name = 'res_sim_traffic_sim_call_date_index'
    
    def _auto_init(self, cr, context={}):
        super(res_sim_traffic, self)._auto_init(cr, context)
        cr.execute('SELECT 1 FROM pg_indexes WHERE indexname=%s',
                   (self._index_name,))
        
        if not cr.fetchone():
            cr.execute('CREATE INDEX {name} ON res_sim_traffic (sim_id, call_date)'.format(name=self._index_name))


In this example::

    res_sim_traffic - is a class name and the name of a table
    sim_id and call_date - columns on which we add index



New Odoo 8.0 API
****************

OpenChatter
===========

::

    # Automatic logging system if mail installed
    _track = {
        'field': {
            'module.subtype_xml': lambda self, cr, uid, obj, context=None: obj[state] == done,
            'module.subtype_xml2': lambda self, cr, uid, obj, context=None: obj[state] != done,
        },
        'field2': {
            ...
        },
    }

.. highlight:: xml

**_track** on the object is used to track events related to a document (an invoice has been paid, an opportunity is won, a task is blocked, ...). Users can follow events, represented by a mail.message.subtype, on any object.

It's different from the **track_visibility** attribute that you can define on a field which is used to track changes on this field. (e.g. Stage : Proposition -> Negociation)

Both **_track** and **track_visibility** produces messages on the document. Your object need to inherit from *mail.thread*.

If an object is inherited from 'mail.thread' then _track is used to send notifications. Therefore 'module.subtype_xml' is the related "Message Subtype". These subtypes have to be declared in XML. Here is an example::

    <record id="subtype_xml" model="mail.message.subtype">
        <field name="name">Relevant Fields</field>
        <field name="res_model">project.issue</field>
        <field name="default" eval="True"/>
        <field name="description">The issue has been closed.</field>
    </record>
    
Then whenever the field "field" is updated, all subtypes ("subtype_xml", "subtype_xml2") of this field are processed.

This means that the related method (in this example: lambda ...) is called and if the result is True, then for all users which follow this object and have checked the subtype a notification is created.

In the user preferences every user can choose whether he/she wants to be updated by email in case of new notifications.

You can also set a mail.message.subtype that depends on an other to act through a relation field. Here is an exemple from crm for Sales Teams crm.case.section using the section_id m2o in crm.lead::

    <record id="mt_lead_won" model="mail.message.subtype">
        <field name="name">Opportunity Won</field>
        <field name="res_model">crm.lead</field>
        <field name="default" eval="False"/>
        <field name="description">Opportunity won</field>
    </record>

    <record id="mt_salesteam_lead_won" model="mail.message.subtype">
        <field name="name">Opportunity Won</field>
        <field name="res_model">crm.case.section</field>
        <field name="parent_id" eval="ref('mt_lead_won')"/>
        <field name="relation_field">section_id</field>
    </record>
    
This allows a user to follow all "Opportunities Won" that are in a specific sales team. The user follow the event "Opportunity Won" on a sales team and he will become automatically follower of all leads/oppotunities of this sales team and _track event.

.. highlight:: python

Example of tracking 'state' field (You can see _track in sale.py)::

    _inherit = ['mail.thread']
    _track = {
        'state': {
            'sale.mt_order_confirmed': lambda self, cr, uid, obj, ctx=None: obj['state'] in ['manual', 'progress'],
            'sale.mt_order_sent': lambda self, cr, uid, obj, ctx=None: obj['state'] in ['sent']
        },
    }

Here, *mt_order_confirmed* & *mt_order_sent* is an ID of record of object mail.message.subtype in sale_data.xml

I also can deside to track a field inside it's definition. For this purpose exists parameter **track_visibility**::

    state = fields.Selection([
        ('draft', _('Draft Quotation')),
        ('assigned', _('Assigned')),
        ('wait_confirm', _("Wait confirmation")),
        ('wait_executed', _("Wait execution")),
        ('wait_payed', _('Wait payment')),
        ('cancel', _('Cancelled')),
        ('done', _('Done')),
    ], 'Status', readonly=True, copy=False, help="Gives the status of the quotation or sales order.",
    track_visibility='onchange', select=True, default='draft')

Running example
---------------

A small my_task model will be used as example to explain how to use the OpenChatter feature. Being simple, it has only the following fields:

    - a name
    - a task responsible
    - a related project

::

    class my_task(osv.osv):
        _name = "my.task"
        _description = "My Task"
        _columns = {
            'name': fields.char('Name', required=True, size=64),
            'user_id':fields.many2one('res.users', string='Responsible',
              ondelete='cascade', required=True, select=1),
            'project_id':fields.many2one('project.project', string='Related project',
              ondelete='cascade', required=True, select=1),
        }

Two-lines feature integration
-----------------------------

Make your object inherit from *mail.thread*::

    class my_task(osv.osv):
        _name = "my.task"
        _description = "My Task"
        # inherit from mail.thread allows the use of OpenChatter
        _inherit = ['mail.thread']

.. highlight:: xml

Use the thread viewer widget inside your form view by using the **mail_thread** widget on the **message_ids** field inherited from *mail.thread*::

    <record model="ir.ui.view" id="my_task_form_view">
        <field name="name">My Task</field>
        <field name="model">my.task</field>
        <field name="priority">1</field>
        <field name="arch" type="xml">
            <form>
                [...]
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers" groups="base.group_user"/>
                    <field name="message_ids" widget="mail_thread"/>
                </div>
            </form>
        </field>
    </record>


Send notifications
------------------

.. highlight:: python

When sending a notification is required in your workflow or business logic, use *mail.thread.message_post()*. It will automatically take care of subscriptions and notifications.

Here is a small example of sending a notification when the do_something method is called::

    def do_something(self, cr, uid, ids, context=None):
        self.do_something_send_note(cr, uid, ids, context=context)
        return res
    
    def do_something_send_note(self, cr, uid, ids, context=None):
        self.message_post(
            cr, uid, ids, _('My subject'),
            _("has received a <b>notification</b> and is happy for it."),
            context=context)


Notifications guidelines
------------------------

    - avoid unnecessary content, swamping users with irrelevant messages will lead to them ignoring all messages
    - use short sentences
    - do not include the document name, this is done by the thread widget
    - use a simple and clean style
        - html tags are supported: use <b> or <em> mainly
        - put key word(s) in bold
        - avoid fancy styles that will break the OpenERP look and feel
        - create a separate method for sending your notification, use clear method names allowing quickly spotting notification code e.g. name notification methods by using the original method name postfixed by _send_note (do_something -> do_something_send_note)


Subscription management
-----------------------

The default subscription behavior is the following:

    - Subscriptions are set up by creating a mail.followers` entry
    - If a user creates or updates a document, they automatically follow it. The corresponding *mail.followers entry* is created
    - If a user explicitly cliks on the document's *Follow* button, they follow the document. The corresponding *mail.followers* entry is created
    - If a user explicitly clicks on the document's *Unfollow* button, they stop following the document. The corresponding *mail.followers* entry is deleted

You should not directly manipulate *mail.followers* entry, if you need to override the default subscription behavior you should override the relevant *mail.thread* methods.


Decorators
==========

https://github.com/nbessi/odoo_new_api_guideline/blob/master/source/decorator.rst

context
=======

To change the context::

    records = self.with_context(new_key='abc').browse(ids)  # current context extended with {'new_key': 'abc'} in records

    records = self.with_context({'new_key': 'abc'}).browse(ids)  # current context replaced by {'new_key': 'abc'} in records


_track proprerty of mail.thread model
=====================================

(From https://www.odoo.com/es_ES/forum/how-to/developers-13/what-is-the-track-proprerty-of-mail-thread-model-used-for-1704)

In the mail.thread model _trackpropety is defined. It as the following doc::

    # Automatic logging system if mail installed
    _track = {
       'field': {
           'module.subtype_xml': lambda self, cr, uid, obj, context=None: obj[state] == done,
           'module.subtype_xml2': lambda self, cr, uid, obj, context=None: obj[state] != done,
       },
       'field2': {
           ...
       },
     }


But his puprose and usage is not really clear.
_track on the object is used to track events related to a document (an invoice has been paid, an opportunity is won, a task is blocked, ...). Users can follow events, represented by a mail.message.subtype, on any object.

It's different from the track_visibility attribute that you can define on a field which is used to track changes on this field. (e.g. Stage : Proposition -> Negociation)

Both _track and track_visibility produces messages on the document. Your object need to inherit from mail.thread.

.. highlight:: xml

If an object is inherited from 'mail.thread' then _track is used to send notifications. Therefore 'module.subtype_xml' is the related "Message Subtype". These subtypes have to be declared in XML. Here is an example::

    <record id="subtype_xml" model="mail.message.subtype">
        <field name="name">Relevant Fields</field>
        <field name="res_model">project.issue</field>
        <field name="default" eval="True"/>
        <field name="description">The issue has been closed.</field>
    </record>

Then whenever the field "field" is updated, all subtypes ("subtype_xml", "subtype_xml2") of this field are processed.

This means that the related method (in this example: lambda ...) is called and if the result is True, then for all users which follow this object and have checked the subtype a notification is created.

In the user preferences every user can choose whether he/she wants to be updated by email in case of new notifications.

You can also set a mail.message.subtype that depends on an other to act through a relation field. Here is an exemple from crm for Sales Teams crm.case.section using the section_id m2o in crm.lead::

    <record id="mt_lead_won" model="mail.message.subtype">
        <field name="name">Opportunity Won</field>
        <field name="res_model">crm.lead</field>
        <field name="default" eval="False"/>
        <field name="description">Opportunity won</field>
    </record>

    <record id="mt_salesteam_lead_won" model="mail.message.subtype">
        <field name="name">Opportunity Won</field>
        <field name="res_model">crm.case.section</field>
        <field name="parent_id" eval="ref('mt_lead_won')"/>
        <field name="relation_field">section_id</field>
    </record>

This allows a user to follow all "Opportunities Won" that are in a specific sales team. The user follow the event "Opportunity Won" on a sales team and he will become automatically follower of all leads/oppotunities of this sales team and _track event.


Other useful model related stuff
********************************

Superuser
=========
.. highlight:: python

Sometimes you need to execute query as Superuser::

    from openerp import SUPERUSER_ID
    chart_templates = chart_obj.browse(cr, SUPERUSER_ID, chart_obj_ids, context)

New API::

    self.sudo(user.id)
    self.sudo() # This will use the SUPERUSER_ID by default
    # or
    self.env[’res.partner’].sudo().create(vals)


cr.execute
==========

- res = cr.dictfetchall()
- res2 = cr.dictfetchone()
- res3 = cr.fetchall()
- res4 = cr.fetchone()

cr.dictfetchall()
    will give you all the matching records in the form of **list of dictionary** containing key, value.

cr.dictfetchone()
    works same way as cr.dictfetchall() except it returns only single record.

cr.fetchall()
    will give you all the matching records in the form of list of tupple.

cr.fetchone()
    works same way as cr.fetchall() except it returns only single record.


In your given query, if you use:
    - cr.dictfetchall() will give you [{'reg_no': 123},{'reg_no': 543},].
    - cr.dictfetchone() will give you {'reg_no': 123}.
    - cr.fetchall() will give you '[(123),(543)]'.
    - cr.fetchone() will give you '(123)'.
