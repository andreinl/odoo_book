Using Sequences
**************

.. highlight:: xml

Create data in ir.sequence table::

    <?xml version="1.0" encoding="utf-8"?> 
    <openerp> 
        <data noupdate="1">
            <record id="seq_type_our_table" model="ir.sequence.type"> 
                <field name="name">Inventory Code</field> 
                <field name="code">our.table</field> 
            </record> 
            <record id="seq_master_item" model="ir.sequence"> 
                <field name="name">Inventory Code</field> 
                <field name="code">out.table</field> 
                <field name="prefix">INV</field> 
                <field name="padding">3</field> 
            </record> 
        </data> 
    </openerp>


This way we create a sequencer which can be identified as seq_master_item of the code (table) out.table.

.. highlight:: python

Now we can find it this way::

    sequence_data_id = self.pool.get('ir.model.data').get_object_reference(cr, uid, 'our_module', 'seq_master_item')


Where our_module is the name of the module we are working on.
Then we can get a new sequence value by doing::

    sequence_data_id = sequence_data_id and sequence_data_id[1] 
    res = self.pool.get('ir.sequence').get_id(cr, uid, sequence_data_id)


There is a simpler way to get a next sequence value::

    res = self.pool.get('ir.sequence').get(cr, uid, 'our.table')


It's OK to use this function if out table has only one sequencer. If there are more than one, then the sequencer with the lowest id will be used. 

.. note:: From the version 6.1, ir_sequence.get() and ir_sequence.get_id() are deprecated. ir_sequence.next_by_code() or ir_sequence.next_by_id() should be used instead.


Function _alter_sequence(self, cr, sequence_id, number_increment, number_next=0)
--------------------------------------------------------------------------------

Permits to set next value of a database counter (Sequenze) to *number_next*::

    self._alter_sequence(cr, sequence_id, sequence.number_increment, number_next=number_next)

.. note:: This function should be used only if sequence.implementation is 'standard'. If implementation is 'no_gap' a value should be written in database

Function _get_number_next_actual(cr, uid, sequence_ids, field_name, args)
-----------------------------------------------------------------------------------

Private function to show next number in sequence. Returns a dictionary with a value for every sequence_id in sequence_ids::

    next_number = self._get_number_next_actual(cr, uid, sequence_ids, 'number_next_actual', None)



