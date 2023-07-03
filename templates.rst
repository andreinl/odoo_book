How to debug templates in Odoo 8.0
**********************************

.. highlight:: xml

You need to use special debug t-tag::

    <p t-esc="l.description" t-debug="pdb" />

    (Pdb) pp qwebcontext


How to disable developer mode or debugging feature in openerp 7.0
*****************************************************************

First of all you need to create one custom module and then follow the steps.

Create file under your_module/static/src/base.xml and write the following code:

.. code-block:: guess

    <templates>
        <t t-extend="UserMenu.about">
            <t t-jquery="a.oe_activate_debug_mode" t-operation="replace"/>
        </t>
    </templates>


.. highlight:: python
Add this file in __openerp__.py::

    'qweb' : [
        "static/src/xml/base.xml",
    ],

 
This code will remove Activate the Developer Mode option for all user.

Following code will disable debug mode (Drop-down box) if user is admin (admin ID must be 1. If not then change session.uid === your_admin_id):

Create one xml file inside your_module/static/src/xml folder and add following code:

.. code-block:: guess

    <templates>
        <t t-extend="ViewManagerAction">
            <t t-jquery="select.oe_debug_view" t-operation="replace">
                <select t-if="widget.session.uid === 1 and widget.session.debug" class="oe_debug_view"/>
            </t>
        </t>
    </templates>

 
Now go to __openerp__.py and add your xml file under qweb.

Like this:

.. code-block:: guess

    'qweb' : [
        "static/src/xml/your.xml",
    ],

 
Restart your server, update your module and refresh the page.

