Working With Workflow
*********************

How It works
============

Table **wkf** (OpenERP 6.1), **workflow** (Odoo 8.0) contains definitions of various workflows.
When a new record is created and there should be a workflow, a new workflow is created in table **wkf_instance**.
Table **wkf_workitem** shows last passage of this workflow.

wkf_instance
------------
- res_type: model
- res_id: model id

wkf_workitem
------------
- inst_id: id of wkf_instance
- act_id: id of wkf_activity - last activity

wkf_activity
------------
- wkf_id: id of wkf table


Examples
========

Example: This will change state of the protocol to close::

    import netsvc    
    wf_service = netsvc.LocalService("workflow")
    
    wf_service.trg_validate(uid, 'project.project', protocol.id, 'act_open', cr)
    wf_service.trg_validate(uid, 'project.project', protocol.id, 'act_disassemble', cr)
    wf_service.trg_validate(uid, 'project.project', protocol.id, 'act_done', cr)
    
Delete and then recreate a Workflow connected to record::

    wf_service.trg_delete(uid, 'sale.order', order.id, cr)
    wf_service.trg_create(uid, 'sale.order', order.id, cr)


API v.8::

    record.signal_workflow('signal')
    self.signal_workflow('action_assign')

Deleting
========
.. highlight:: xml

Sometimes you may need to delete an activity, transition or even workflow. A tag **delete** will do the trick::

    <delete  model="workflow" id="sale.wkf_sale" />
    <delete  model="workflow.activity" id="sale.act_draft" />


Actions
=======

Actions can be defined in workflow and should contain valid python code::

    <field name="kind">function</field>
    <field name="action">write({'state': 'wait_confirm'})</field>

If we need more logic, we can define action inside python and place a call for the function inside workflow::

    <field name="kind">function</field>
    <field name="action">action_sent()</field>
    
.. highlight:: python

A function should look like this::

    def action_sent(self, cr, uid, ids, context=None):
        self.write(cr, uid, ids, {'state': 'wait_confirm'})
        return {}

.. note:: Be careful if you have more workflows defined inside the same module. Record ids should be **unique** inside the model.
