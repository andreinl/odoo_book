Actions
*******

Action files are usually placed inside *wizard* directory.

Action called from Actions menu
===============================

.. highlight:: xml

Calling a function **upgrade_modules()**::

    <record id="action_upgrade_modules" model="ir.actions.server">
        <field name="name">Upgrade modules</field>
        <field name="model_id" ref="model_ir_module_module"/>
        <field name="state">code</field>
        <field name="code">self.upgrade_modules(cr, uid, context=context)</field>
    </record>

    <record id="menu_upgrade_modules" model="ir.values">
        <field name="object" eval="True" />
        <field name="name">Upgrade Modules Values</field>
        <field name="key2">client_action_multi</field>
        <field name="value" eval="'ir.actions.server,%d'%action_upgrade_modules" />
        <field name="key">action</field>
        <field name="model">ir.module.module</field>
    </record>

.. highlight:: python

Calling a wizard (OpenERP 6.0)::

    from osv import fields, osv
    from tools.translate import _
    

    class wizard_update_invoice(osv.osv_memory):
        _name = "wizard.update.invoice"
        _description = 'Update invoice'
        
        _columns = {
            'client_name': fields.char("Name", size=35),
        }
        
        _defaults = {
            'client_name': 'Mario',
        }
        
        def update_invoice(self, cr, uid, wizard_update_invoice_ids, context=None):
            if context is None:
                context = {}
            
            # Get selected invoices:
            invoice_ids = context and context.get('active_ids',[]) or []
            
            # Get values from view connected to memory table:
            update = self.browse(cr, uid, wizard_update_invoice_ids[0])
            client_name = update.client_name
            
            ## Do something ....
            
            # Close window
            return {'type': 'ir.actions.act_window_close'}
                
    wizard_update_invoice()


.. highlight:: xml

This is a corrisponding XML file. It will create an action menu "Update Invoice" inside the right panel::

    <?xml version="1.0" encoding="utf-8"?>
    <openerp>
        <data>
            <record id="view_wizard_update_invoice" model="ir.ui.view">
                <field name="name">wizard.update.invoice.form</field>
                <field name="model">wizard.update.invoice</field>
                <field name="type">form</field>
                <field name="arch" type="xml">
                    <form string="Update Invoice created from Protocol">
                        <field name="client_name" />
                        <separator string="" colspan="4" />
                        <group colspan="4" col="6">
                            <button icon="gtk-cancel" special="cancel"
                                string="Cancel" />
                            <button icon="gtk-save" string="Update Invoice"
                                name="update_invoice" type="object" groups="account.group_account_invoice"/>
                        </group>
                   </form>
                </field>
            </record>
            
            <record id="action_wizard_update_invoice" model="ir.actions.act_window">
                <field name="name">Update Invoice</field>
                <field name="res_model">wizard.update.invoice</field>
                <field name="view_type">form</field>
                <field name="view_mode">tree,form</field>
                <field name="view_id" ref="view_wizard_update_invoice"/> 
                <field name="target">new</field>
            </record>
            
            <record id="menu_update_invoice" model="ir.values">
                <!--<field name="model_id" ref="model_account_invoice" />-->
                <field name="object" eval="True" />
                <field name="name">Invoice Update Menu</field>
                <field name="key2">client_action_multi</field>
                <field name="value" eval="'ir.actions.act_window,' + str(ref('action_wizard_update_invoice'))" />
                <field name="key">action</field>
                <field name="model">account.invoice</field>
            </record>
        </data>
    </openerp>    

.. note:: Don't forget to add import of a python script to *wizard/__init__.py* and xml file to *__openerp__.py*

In Odoo v.8.0 lateral menu moved on top in the center.

This code adds menu **Sync Categories** to the central top menu. When selected it calls function **sync_all** of the product.public.category::

    <record id="action_category_sync" model="ir.actions.server">
        <field name="name">Sync Categories</field>
        <field name="model_id" ref="model_product_public_category"/>
        <field name="state">code</field>
        <field name="code">
            if context.get('active_model') == 'product.public.category':
                self.sync_all(cr, uid, [], context=context)
        </field>
    </record>

    <record id="category_sync" model="ir.values">
        <field eval="'client_action_multi'" name="key2"/>
        <field eval="'product.public.category'" name="model"/>
        <field name="name">Sync Categories</field>
        <field eval="'ir.actions.server,%d'%action_category_sync" name="value"/>
    </record>

Changing destination view attributes
------------------------------------

To open a tree view with "editable" = Top we can pass 'set_editable': True in the context::

    <record id="open_gtd_task" model="ir.actions.act_window" >
        <field name="name">My Tasks</field>
        <field name="res_model">project.task</field>
        <field name="search_view_id" ref="view_task_gtd_search"/>
        <field name="context">{'set_editable': True}</field>
    </record>


Dialogs
=======

.. highlight:: python

OpenERP 7 comes with two client actions that will result in a Growl-like message in the user’s browser: “action_warn” and “action_info.” They both take three parameters: “title,” “text,” and “sticky.” The first two are the title text and the message text, respectively, as strings. The third is a boolean indicating whether the message should remain on the screen until the user closes it by clicking the “x” in the corner of it. Setting this to false will result in the message fading away on its own after a few seconds.

To trigger these client actions from a server action, just have the server action return a dictionary like this one::

    return {
        'type': 'ir.actions.client',
        'tag': 'action_warn',
        'name': 'Failure',
        'params': {
           'title': 'Postage Cancellation Failed',
           'text': 'Shipment is outside the void period.',
           'sticky': True
        }
    }

    return {
        'type': 'ir.actions.client',
        'tag': 'action_info',
        'name': _('Growl'),
        'params': {
            'title': _('Grr...'),
            'text': _('I am growling at you!'),
            'sticky': False
        }
    }
