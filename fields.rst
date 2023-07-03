Fields
******

All fields
==========

All fields can be setted in readonly for all but some states::

    _columns = {
        'order_line': fields.one2many('sale.order.line', 'order_id', 'Order Lines', readonly=True, states={
            'draft': [('readonly', False)],
            'wait_technical_validation': [('readonly', False)],
            'wait_manager_validation': [('readonly', False)]}
        ),
    }

From Odoo 8.0:
--------------
A value of select="1" means that the field is included in the basic search, and a value of select="2" means that it is in the advanced search.

index -- whether the field is indexed in database (boolean, by default False)


Functional field
=================

Complete example::

    class sale_order_line(osv.osv):
        _inherit = "sale.order.line"

        def _with_bom(self, cr, uid, ids, field_name, arg, context):
            result = {}
            
            lines = self.browse(cr, uid, ids)
            
            for line in lines:
                if line.product_id.supply_method == 'produce':
                    result[line.id] = True
                else:
                    result[line.id] = False
            
            return result

        _columns = {
            'with_bom': fields.function(_with_bom, method=True, type='boolean')
        }

    sale_order_line()


If function type is *'selection'* parameter selection should be set::

    DIRECTIONS = [
        ('in', 'IN'),
        ('out', 'OUT')
    ]
    
    ...
    
    def _get_direction(self, cr, uid, ids, field_name, arg, context):
        result = {}
        lines = self.browse(cr, uid, ids)

        for line in lines:
            if line.type.move:
                result[line.id] = line.type.move
            else:
                result[line.id] = False
        return result
    
    _columns = {
        'move': fields.function(_get_direction, string='Move', type='selection', selection=DIRECTIONS, method=True, help="Incoming or Outgoing Letter"),
    }

.. note:: Don't use ``widget="selection"`` attribute in the fields view. It will generate **AttributeError: 'NoneType' object has no attribute '_name_search'** error

If function type is *'many2one'*, you should set *obj* parameter::

    def _get_stock_location(self, cr, uid, ids, field_name, args, context=None):
        move_obj = self.pool['stock.move']
        
        result = {}
        
        for asset in self.browse(cr, uid, ids, context):
            move_ids = move_obj.search(cr, uid, [('product_id', '=', asset.asset_product_id.product_product_id.id), ('prodlot_id', '=', asset.serial_number.id)], order='date desc', limit=1)
            
            if move_ids:
                move = move_obj.browse(cr, uid, move_ids[0], context)
                result[asset.id] = move.location_dest_id.id
            else:
                result[asset.id] = False
            
        return result

    _columns = {
        'stock_location_id': fields.function(_get_stock_location, "Current Stock Location", method=True, obj='stock.location', type='many2one')
    }


If the type of a function is *'many2many'*, the result is a dictionary where every line has a list of ids::

    def _compute_lines(self, cr, uid, ids, name, args, context=None):
        result = {}
        for statement in self.browse(cr, uid, ids, context=context):
            src = []
            lines = []
            if statement.move_id:
                for m in statement.move_id.line_id:
                    temp_lines = []
                    if m.reconcile_id:
                        temp_lines = map(lambda x: x.id, m.reconcile_id.line_id)
                    elif m.reconcile_partial_id:
                        temp_lines = map(lambda x: x.id, m.reconcile_partial_id.line_partial_ids)
                    lines += [x for x in temp_lines if x not in lines]
                    src.append(m.id)

            lines = filter(lambda x: x not in src, lines)
            result[statement.id] = lines
        return result

    _columns = {
        'payment_ids': fields.function(_compute_lines, relation='account.move.line', type="many2many", string='Payments'),
    }

One of the functional field parameters is *'multi'*. All fields with the same multi name will be calculated in a single function call.

Example with **multi=False**. Function will return a dictionary where every record id has single value:: 

    def _check_picking_done(self, cr, uid, ids, filed_name, arg, context=None):
        '''
        'picking_id_name': 'in_picking_id', 'out_picking_id', 'in_picking_producer_id', 'out_picking2producer_id'
        '''
        res = {}.fromkeys(ids, False)
        if not len(ids) or not arg.get('picking_id_name', False):
            return res
        
        for order in self.read(cr, uid, ids, [arg['picking_id_name'], 'state']):
            if order.get(arg['picking_id_name'], False):
                picking = self.pool['stock.picking'].read(cr, uid, order[arg['picking_id_name']][0], ['state'], context=context)
                if picking and picking['state'] == 'done':
                    res[order['id']] = True
        return res

    _columns = {
        'inward_ok': fields.function(_check_picking_done, arg={'picking_id_name': 'in_picking_id'}, method=True, multi=False, type="boolean", string="Inward ok?",
                                     store={'stock.picking': (_check_picking_state, ['state'], 10)}),
        'outward_producer_ok': fields.function(_check_picking_done, arg={'picking_id_name': 'out_picking2producer_id'}, method=True, multi=False, type="boolean", string="Sent to Producer?",
                                               store={'stock.picking': (_check_picking_state, ['state'], 10)}),
        'inward_producer_ok': fields.function(_check_picking_done, arg={'picking_id_name': 'in_picking_producer_id'}, method=True, multi=False, type="boolean", string="Recived from Producer?",
                                              store={'stock.picking': (_check_picking_state, ['state'], 10)}),
    }


Example with **multi=True**. Function will return a dictionary where every record id has a dictionary of values. A key in this dictionary is the name of a field. (In this particular example field_name == picking_repair_field_map.values())::
    
    def _check_picking_done(self, cr, uid, ids, field_name, arg, context=None):
        picking_repair_field_map = {
            'in_picking_id': 'inward_ok',
            'out_picking_id': 'is_delivered',
            'in_picking_producer_id': 'inward_producer_ok',
            'out_picking2producer_id': 'outward_producer_ok'
        }
        
        res = {}.fromkeys(ids, dict([(field, False) for field in picking_repair_field_map.values()]))
        if not ids:
            return res
        
        repair_fields = picking_repair_field_map.keys()
        repair_fields.append('state')
        for order in self.read(cr, uid, ids, repair_fields):
            for field in picking_repair_field_map.keys():
                if order.get(field, False):
                    picking = self.pool['stock.picking'].read(cr, uid, order[field][0], ['state'], context=context)
                    if picking and picking['state'] == 'done':
                        res[order['id']][picking_repair_field_map[field]] = True

        return res

    _columns = {
        'inward_ok': fields.function(_check_picking_done, method=True, multi='picking_done', type="boolean", string="Inward ok?",
                                     store={'stock.picking': (_check_picking_state, ['state'], 10)}),
        'outward_producer_ok': fields.function(_check_picking_done, method=True, multi='picking_done', type="boolean", string="Sent to Producer?",
                                               store={'stock.picking': (_check_picking_state, ['state'], 10)}),
        'inward_producer_ok': fields.function(_check_picking_done, method=True, multi='picking_done', type="boolean", string="Recived from Producer?",
                                              store={'stock.picking': (_check_picking_state, ['state'], 10)}),
        'is_delivered': fields.function(_check_picking_done, method=True, multi='picking_done', type="boolean", string="Delivered to Customer",
                                        store={'stock.picking': (_check_picking_state, ['state'], 10)}),
    }


Search parameter
----------------

.. note:: To use domain filter, you need define search function for functional field.


Next example shows how to write e search function for a functional field.::

    def _search_direction(self, cr, uid, obj, name, args, context):
        if not args:
            return []
        
        for search in args:
            if search[0] == 'move':
                search_key = args[0][2]
                query = """SELECT res_letter.id FROM res_letter
                    LEFT JOIN letter_type 
                    ON res_letter.type = letter_type.id
                    WHERE move='{0}'""".format(search_key)
                cr.execute(query)
                letters = cr.fetchall()
                res = [letter[0] for letter in letters]
                return [('id', 'in', res)]
        return []
        
    _columns = {
       'move': fields.function(_get_direction, string='Move', type='selection', fnct_search=_search_direction, selection=DIRECTIONS, method=True, help="Incoming or Outgoing Letter"),
    }         


More complex example. In this case function shows a relational value, so we need a way to write a custom query for every single model. We are achieving it with a dictionary model_search_field::

    def search_location(self, cr, uid, obj, name, args, context):
        if not args:
            return
        
        locations = []
        res = []
        
        model_search_field = {
            'asset.asset': {'field': 'track_no', 'field_name': 'location'},
            'asset.move': {'field_name': 'dest_location'},
            'project.project': {'query_start': """SELECT project_project.id FROM {model} LEFT JOIN account_analytic_account 
                ON account_analytic_account.id = project_project.analytic_account_id """, 'field': 'name'},
            'res.partner': {'field': 'name'},
            'hr.employee': {'query_start': """SELECT hr_employee.id FROM {model} LEFT JOIN resource_resource 
                ON resource_resource.id = hr_employee.resource_id """, 'field': 'name'},
            'res.partner.contact': {'field': 'name'}, # may be in the future we should add first_name also
            'stock.location': {'field': 'name'},
            'res.car': {'field': 'plate'}, # search car by the plate
            'project.place': {'query_start': """SELECT project_place.id FROM {model} LEFT JOIN res_partner_address 
                ON res_partner_address.id = project_place.address_id """, 'field': 'res_partner_address.name'},
            'project.plant': {'query_start': """SELECT project_plant.id FROM {model} LEFT JOIN res_partner_address 
                ON res_partner_address.id = project_plant.address_id """, 'field': 'res_partner_address.name'},
        }
        
        query_middle = "WHERE {model}.id = '{row_id}' AND "

        wanted_values = args[0][2].split(',')
        field_name = model_search_field[self._name]['field_name']
        
        # Get all fields among which we should search:
        cr.execute("""SELECT {field_name} FROM {table}
                        WHERE {field_name} IS NOT NULL
                        GROUP BY {field_name}""".format(table=self._name.replace('.', '_'), field_name=field_name))
        pretenders = cr.fetchall()
        pretenders = [p[0] for p in pretenders]
        
        for pretender in pretenders:
            model, row_id = pretender.split(',')
            if model in model_search_field.keys():
                if len(wanted_values) > 1:
                    query_ends = ["{0} ILIKE '%{1}%'".format(model_search_field[model]['field'], v.strip()) for v in wanted_values]
                    query_end = ' OR '.join(query_ends)
                else:
                    query_end = "{0} ILIKE '%{1}%'".format(model_search_field[model]['field'], wanted_values[0].strip())
                
                if model_search_field[model].has_key('query_start'):
                    query_start = model_search_field[model]['query_start']
                else:
                    query_start = "SELECT id FROM {model} "
                
                query = query_start.format(model=model.replace('.', '_')) + query_middle.format(model=model.replace('.', '_'), row_id=row_id) + query_end

                cr.execute(query)
                locations += ['{0},{1}'.format(model, r[0]) for r in cr.fetchall()]
        
        for location in locations:
            #res += self.search(cr, uid, [(field_name, 'like', '{location}'.format(location=location))])
            res += self.search(cr, uid, [(field_name, '=', '{location}'.format(location=location))])
        return [('id', 'in', res)]

    _columns = {
        'location_name': fields.function(get_relational_value, arg={'field_name': 'location'}, fnct_search=search_location, method=True, type="char", string="Current Location"),
    }


Write parameter
---------------

Sometimes you need to write to functional field, for example if field is used to dupplicate another field, so it can be shown in more places. To achieve it you should set `fnct_inv` parameter::

    class res_partner(osv.osv):
        _inherit = "res.partner"
        
        def _set_fiscalcode(self, cr, uid, ids, field_name, field_value, arg, context):
            self.write(cr, uid, ids, {'fiscalcode': field_value})
            return True
        
        def _get_fiscalcode(self, cr, uid, ids, field_name, arg, context):
            if not ids:
                return False
            
            result = {}
            
            partners = self.browse(cr, uid, ids)
            for partner in partners:
                result[partner.id] = partner.fiscalcode
                
            return result
        
        _columns = {
            'pec': fields.related('address', 'pec', type='char', size=64, string='PEC'),
            'cf': fields.function(_get_fiscalcode, fnct_inv=_set_fiscalcode, string=_("Fiscal code"), type='char', method=True),
            'individual': fields.boolean(_('Individual')),
        }
    res_partner()


compute_function_name
---------------------

(New API)::

    task_ids = fields.Many2many('project.task', string=_('Tasks'), compute='_compute_tasks')

    @api.one
    def _compute_tasks(self):
        tasks = self.env['project.task'].search([('sale_line_id', 'in', self.order_line.ids)])
        if tasks:
            self.task_ids = tasks.ids


Selection field
===============

Complete example of the selection field with dynamic content::
    
    def _get_bank_account_ids(self, cr, uid, context=None):
        bank_obj = self.pool.get('res.partner.bank')
        result = []
        bank_ids = bank_obj.search(cr, uid, [('partner_id', '=', 1)])
        banks = bank_obj.browse(cr, uid, bank_ids)
        for bank in banks:
            result.append((bank.id, bank.bank.name))
        return result
    
    _columns = {
        'bank': fields.selection(_get_bank_account_ids, 'Banca', required=True),
    }

.. note:: Index of the selection should be a string. So ((1, 'A'), (2, 'B'), (3, 'C')) will not work. The correct way is: (('1', 'A'), ('2', 'B'), ('3', 'C'))


Extending fields.selection options without overwriting them
-----------------------------------------------------------

(https://www.odoo.com/forum/how-to/developers-13/how-to-extend-fields-selection-options-without-overwriting-them-21529)

Say you want to modify the selection field type of the product categories. Excerpt of the code from the product addon, that you can't modify::

    class stock_picking(orm.Model):
        _name = "stock.picking"
        _description = "Picking List"
        _columns = {
            # <snip>
            'state': fields.selection([
                ('draft', 'New'),
                ('auto', 'Waiting Another Operation'),
                ('confirmed', 'Waiting Availability'),
                ('assigned', 'Ready to Process'),
                ('done', 'Done'),
                ('cancel', 'Cancelled'),
                ], 'State', readonly=True, select=True,
                help="* Draft: not confirmed yet and will not be scheduled until confirmed\n"\
                     "* Confirmed: still waiting for the availability of products\n"\
                     "* Available: products reserved, simply waiting for confirmation.\n"\
                     "* Waiting: waiting for another move to proceed before it becomes automatically available (e.g. in Make-To-Order flows)\n"\
                     "* Done: has been processed, can't be modified or cancelled anymore\n"\
                     "* Cancelled: has been cancelled, can't be confirmed anymore"),
        }

In your module you need to alter the field in the _columns in the __init__ of the model::

    class stock_picking(orm.Model):
        _inherit = 'stock.picking'
        
        def __init__(self, pool, cr):
            super(stock_picking, self).__init__(pool, cr)
            option = ('rented', 'Rented')
            state_selection = self._columns['state'].selection
            if option not in state_selection:
                state_selection.append(option)

    
one2many field
==============

.. note:: New in version 6.1

One2Many lines are treated as sub-records of their parent record, so they are supposed to be saved along with the rest of the record data, atomically (in a single RPC call).

In this sense, the save button on the sub-record does not directly call any method on the server, it simply saves the changes in a local cache in your browser. The real call to create (if this is a new record) or write (if the record is being updated) will only be done when you click on the main Save button of the parent record form afterwards.

At this point, the value of the line will be passed in the map of values provided to create/write, within a list of One2Many commands.

.. note:: It's important to pass parent id to subrecord.

Example::

    _columns = {
        'payment_delay_ids': fields.one2many('account.payment.delay', 'partner_id', 'Payment Delays')
    }

.. highlight:: xml

::

    <field name="payment_delay_ids" nolabel="1" context="{'partner_id': active_id}">
        <tree editable="bottom">
            <field name="partner_id" invisible="True" />
            <field name="month" />
            <field name="delay" />
        </tree>
    </field>

Map of values is organized as array of lists. The first element of the list determines the state of value inside array:

    - [**0**, False, {'value1': x, 'value2': y}] - New added line
    - [**1**, 56, {'value2': z}] - Modified line    
    - [**2**, 56, False] or [**2**, False, {'value1': x, 'value2': y}] - Deleted lines
    - [**4**, 56, False] - Line from database. The id of the line is 56

.. highlight:: python

many2one field
==============

Example::

    _columns = {
        'product_id': fields.many2one('product.product', 'Product', required=True),
        'partner_id': fields.many2one('res.partner', 'Supplier', domain="[('supplier', '=', True)]")
    }


many2many field
===============

Example::

    _columns = {
        'work_order_default_task_ids': fields.many2many('project.task', string='Default Work Order Tasks')
    }


This will create `project_task_res_company_rel` relational table

.. note:: A strange syntax is needed for backward compatibility

From v.8 there is a new syntax::

    supplier_ids = openerp.fields.Many2many('res.partner', string=_('Suppliers'), domain="[('supplier', '=', True)]")


Filtering can be complicated on many2many field. In this example we have a wizard and we need to give a possibility to select from related records. This can be achieved with the help of the second field. As we want that only one record is selected we use many2one field::

    class crm_case_categ(openerp.models.Model):
        _inherit = 'crm.case.categ'
        
        supplier_ids = openerp.fields.Many2many('res.partner', string=_('Suppliers'), domain="[('supplier', '=', True)]")


    class crm_category(orm.TransientModel):
        _name = 'crm.make.sale.category'
        
        _columns = {
            'name': fields.related('categ_id', 'name', type='char', string=_('Name'), store=False, readonly=True),
            
            'categ_id': fields.many2one('crm.case.categ', 'Services'),
            'partner_id': fields.many2one('res.partner', 'Supplier', create=False),
            'supplier_ids': fields.related('categ_id', 'supplier_ids', string=_('Supplier m2m'), relation='res.partner', type="many2many", store=False),
            
            'make_sale_id': fields.many2one('crm.make.sale', 'make sale'),
        }


.. highlight:: xml

And our view will be organized this way::

    <field name="categ_ids" colspan="4">
        <tree string="Required Services" editable="top" create="false">
            <field name="name" />
            <field name="partner_id" create="False" edit="False" domain="[('id', 'in', supplier_ids[0][2])]" />
            <field name="supplier_ids" invisible="True" />
        </tree>
    </field>


.. highlight:: python


We also can move domain inside field definition and use default_get to generate a supporting field::

    truck_info_id = fields.Many2one('broker.truck.info', _('Truck info'), required=True,
                                    domain="[('id', 'in', truck_info_ids[0][2])]")
    truck_info_ids = fields.Many2many('broker.truck.info')

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


date field
==========

Syntax::

    fields.date('Field Name' [, Optional Parameters]),


datetime field
==============

Allows to store a date and the time of day in the same field.

Syntax::

    fields.datetime('Field Name' [, Optional Parameters]),


Reference field
===============

This is the less documented field. Value of a field consists of two values separated by ",": model name and row id::

    class stock_move(orm.Model):
        _inherit = "stock.move"
        
        def _links_get(self, cr, uid, context=None):
            obj = self.pool.get('res.request.link')
            ids = obj.search(cr, uid, [], context=context)
            res = obj.read(cr, uid, ids, ['object', 'name'], context)
            return [(r['object'], r['name']) for r in res]

        _columns = {
            'origin_document': fields.reference("Origin Document", selection=_links_get, size=None)
        }

We also can use an OpenERP function::

    _columns = {
        'origin': fields.reference(_('Reference'), selection=base.res.res_request._links_get, size=None),
    }

Once there was a function *referencable_models*. Seems that it still works in 8.0, but not in 6.1. It workes the same as *_links_get*::

    _columns = {
        'origin_document': fields.reference(_("Origin Document"), selection=openerp.addons.base.res.res_request.referencable_models, size=None)
    }
    
.. highlight:: xml

This function will return values from the table 'res_request_link'. It can be a good idea to verify if the table we want to use as a reference is present in this table. If not we should add it this way::

    <?xml version="1.0" encoding="utf-8"?>
    <openerp>
        <data noupdate="1">
            <!-- Requests Links -->
            <record id="req_link_crm_lead" model="res.request.link">
                <field name="name">CRM Lead</field>
                <field name="object">crm.lead</field>
            </record>
        </data>
    </openerp>

.. highlight:: python

In new Odoo 8.0 API syntax has changed, the new way to reference is like this::

    @api.multi
    def get_referencable_models(self):
        return addons.base.res.res_request.referencable_models(self, self._cr, self._uid, self._context)
        
    origin = fields.Reference(get_referencable_models, _('Origin'))
    

To write to this field we should compose a value by ourselves::

    model_obj.write(cr, uid, ids, {
        'origin_document': "sale.order, {0}".format(merge.sale_id.id)
    })

.. highlight:: python

Related field
=============

This field permits to refer to the relation of a relation. For example we have a stock_picking which has a sale_id field which has project_id field (picking -> sale_id -> project_id)::

    class stock_picking(osv.osv):
        _inherit = 'stock.picking'

        _columns = {
            'sale_analytic_account': fields.related('sale_id', 'project_id', type='many2one', relation='account.analytic.account', string=_('Sale Analytic Account'), store=False)
        }


Binary field
============

Binary field contains binary data encrypted base64.

To be able to download data we should create 2 fields: one for the name of a file and the second with data::

    order_filename = fields.Char()
    order_file = fields.Binary('File', readonly=True)

    @api.multi
    def create_purchase_order(self):
        file_data = StringIO()

        for sale_order in self.order_ids:
            file_data.write(sale_order.name)

        out = file_data.getvalue()

        self.order_filename = 'purchase_order_{}.txt'.format(self.name)
        self.order_file = out.encode("base64")


.. highlight:: xml

And place fields in a view::

    <field name="order_filename" invisible="True"/>
    <field name="order_file" filename="order_filename"/>


Char field
==========
If you need to store passwords and don't want it to be visible you can define **password** field in view::

    <field name="rpc_password" password="True" />


HTML field
==========
If you need to use HTML inside your text you can define it in view::

    <field name="description" widget="html"/>

.. highlight:: python

or define the HTML field in the model::

    description = fields.Html('Description')

.. note:: Tested on Odoo 8.0


Constraints and SQL Constraints
===============================

We can impose that some fields or combinations of fields are unique. This can be done with constraints.

In this example combination of 'name' + 'product_id' is unique::

    def _check_name_unique(self, cr, uid, ids, context=None):
        if len(ids) == 1:
            lot = self.browse(cr, uid, ids[0], context)
            if lot.product_id.lot_split_type == 'single':
                lot_ids = self.search(cr, uid, [('product_id', '=', lot.product_id.id), ('name', '=', lot.name)])
                if len(lot_ids) == 1:
                    return True
                else:
                    print '####### Duplicate serial number ########'
                    return False
            else:
                return True
        
        return False
    
    _constraints = [
        (_check_name_unique, _('Duplicate serial number'), ['name', 'product_id'])
    ]
    

This can also be achieved at database level::

    _sql_constraints = [('code_uniq', 'unique(code)', 'Code must be unique!')]
    

Verify that two fields always differs one from another::

    _sql_constraints = [
        ('main_id', 'CHECK (sequence_main_id != sequence_id)',
            'Main Sequence must be different from current !'),
    ]

Odoo New API
------------

In New API a decorator @api.constrains should be used::

    from openerp.exceptions import ValidationError

    @api.one
    @api.constrains('start_point', 'end_point'):
    def _check_itinerary_unique(self):
        if self.search([('start_point', '=', self.start_point), ('end_point', '=', self.end_point)]):
            raise ValidationError("This itinerary is already exists")

Defaults
========

Sometimes it is usefull to set default values. This can be done in 3 modes:

Using `create()` function
-------------------------
::

    def create(self, cr, uid, vals, context={}):
        if not 'color' in vals:
            vals['color'] = 'yellow'
        return super(my_class, self).create(cr, uid, vals, context=context)


Using `_defaults` dictionary
----------------------------
 
This is the most used way to set default values::

    def _get_default_address(self, cr, uid, field, context=None):
        if context.get('default_place_id', False):
            places = self.pool['project.place'].read(cr, uid, context['default_place_id'], [field])
            if places:
                return places[field]    
        return
        
        _defaults = {
            'code': '/',  # lambda self, cr, uid, context: self.pool.get('ir.sequence').get(cr, uid, 'project.plant'),
            'type': 'plant',
            'street': lambda self, cr, uid, context: self._get_default_address(cr, uid, 'street', context),
            'street2': lambda self, cr, uid, context: self._get_default_address(cr, uid, 'street2', context),
        }

.. note:: If we need set default *date* or *datetime* field, **lambda** function should be used. If not, the field will get the time of a server restart.

The **wrong** way::

    _defaults = {
        'date': datetime.now().strftime(DEFAULT_SERVER_DATETIME_FORMAT)
    }
    
The **right** way::

    _defaults = {
        'date': lambda *a: datetime.now().strftime(DEFAULT_SERVER_DATETIME_FORMAT)
    }

There is also a special function::

    _defaults = {
        'order_date': fields.date.context_today
    }

.. note:: This function returns *date* for v.6.1, but *datetime* for v.8.


Using `default_XXX` in xml
--------------------------
  
If we want to use active record of a "parent" in a "child", the way to pass parent id is this:
    
.. code-block:: guess
    
    <field name="plant_ids" colspan="4" nolabel="1" context="{'default_place_id': active_id}"/>


`place_id` is a field of a plants table.
 

Renaming a column
=================

If you need to rename a column without loosing its values it can be done using an attribute **oldname**::

    _columns = {
        'db_datas': fields.binary('Data', oldname='datas')
    }
 
