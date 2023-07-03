Debugging
*********

Investigating RPC calls
=======================

* On the **server-side**, you can start the openerp-server process with the *--log-level=debug_rpc_answer* parameter to get detailed logging of all RPC calls
* On the web **client side**, you can simply use your web browser's debugger to watch all RPC calls (in the Network monitoring tab). Most JSON-RPC calls correspond to regular OpenERP ORM method calls, and are easily readable once you're familiar with the OpenERP RPC API.


Logging
=======

If we just want to see output in terminal during development we can use *print*::

    print something

For a more serious logging we should use logging module::
    
    import logging
    _logger = logging.getLogger(__name__)
    _logger.setLevel(logging.DEBUG)
    
    _logger.debug(u"This is an important debug message")


.. note:: netsvc.Logger() is deprecated in OpenERP 6.1 but it should be used for the earlier versions.

::

    import netsvc
    LOGGER = netsvc.Logger()
    
    LOGGER.notifyChannel(self._name, netsvc.LOG_DEBUG, 'Something happend')
    
Using python debugger
=====================

for advanced debugging of the server values python pdb module should be used::

    import pdb
    ## Set the break point:
    pdb.set_trace()
    

The main commands to navigate when in debugger mode

c(ont(inue))
    Continue execution, only stop when a breakpoint is encountered.
l(ist) [first[, last]]
    List source code for the current file. Without arguments, list 11 lines around the current line or continue the previous listing. With one argument, list 11 lines around at that line. With two arguments, list the given range; if the second argument is less than the first, it is interpreted as a count.
s(tep)
    Execute the current line, stop at the first possible occasion (either in a function that is called or on the next line in the current function).
n(ext)
    Continue execution until the next line in the current function is reached or it returns. (The difference between next and step is that step stops inside a called function, while next executes called functions at (nearly) full speed, only stopping at the next line in the current function.)
a(rgs)
    Print the argument list of the current function.
p *expression*
    Evaluate the expression in the current context and print its value.

.. Note:: print can also be used, but is not a debugger command — this executes the Python print statement.

pp *expression*
    Like the p command, except the value of the expression is pretty-printed using the pprint module.


Investigate fields of an object
===============================

::

    >>> obj_X = self.pool.get('account.invoice')
    >>> print obj_X.fields_get(cr, uid, context=context)
    {'state': {'selectable': True, 'readonly': True, 'selection': [('open', u'Open'), ('done', u'Done'), ('cancel', u'Cancel')], 'type': 'selection', 'string': 'State Alert'}}



Profiling
=========

::

    import cProfile, pstats, StringIO
    pr = cProfile.Profile()
    pr.enable()
    # ... do something ...
    pr.disable()
    s = StringIO.StringIO()
    sortby = 'cumulative'
    ps = pstats.Stats(pr, stream=s).sort_stats(sortby)
    ps.print_stats()
    print s.getvalue()


Profiling with profilehooks
---------------------------

::

    $ pip install profilehooks


::

    from profilehooks import profile

    @profile
    def my_function(args, etc):
        pass


Profiling with line_profiler
----------------------------

* https://github.com/rkern/line_profiler
* https://www.huyng.com/posts/python-performance-analysis


timeit
------

.. Note:: This example works only in IPython (ipython)

::

    >>> import timeit
    >>> s = 'heheheheheh'        
    >>> %timeit s.lower().startswith('he')
    10000 loops, best of 3: 41.3 us per loop


A list of installed modules (Odoo 8.0)
======================================

Go to http://<hostname>:8069/website/info. If you are logged in you got more extensive info.


.. _inspect-label:

Use inspect to get the caller's info from callee
================================================

Use *inspect* python module::

    import inspect

    def hello():
        frame, filename, line_number, function_name, lines, index = inspect.getouterframes(inspect.currentframe())[1]
        print(frame, filename, line_number, function_name, lines, index)

    def hello2():
        frame, filename, line_number, function_name, lines, index = inspect.stack()[1]
        print(frame, filename, line_number, function_name, lines, index)


    hello()
    # (<frame object at 0x8ba7254>, '/home/unutbu/pybin/test.py', 10, '<module>', ['hello()\n'], 0)

