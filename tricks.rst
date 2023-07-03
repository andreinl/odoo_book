Python tricks
*************

Delete duplicate elements from list
===================================

::

    >>> ls = [1, 2, 3, 3, 3]
    >>> list(set(ls))
    [1, 2, 3]


Using lambda function to set default value
==========================================

::

    _defaults = {
        'with_bom': lambda self, cr, uid, context: self._with_bom(cr, uid, context),
    }
    

Shorthands
==========

::

    >>> a = {'name': 'Andrei', 'surname': 'Levin'}
    >>> 'name' in a
    True


This is equivalent to::

    >>> a.has_key('name')
    True


How to concatenate two dictionaries
===================================

    >>> v = {1: 2, 3: 4}
    >>> v.update({'a': 'b', 'c': 'g'})
    >>> print v
    {'a': 'b', 1: 2, 'c': 'g', 3: 4}
    
    
How to concatenate two lists
============================
    
    >>> x = [1, 2, 3]
    >>> x.extend([4, 5])
    >>> print (x)
    [1, 2, 3, 4, 5]


for loop with index
===================
::

    for count, value in enumerate(my_list, start=0)
        print count, value


Using reduce()
==============

    >>> location_ids = [11, 12]
    >>> reduce(lambda x, y: x + ',' + str(y), location_ids[1:], str(location_ids[0]))
    '11, 12'

Dict Comprehensions
===================
::

    def _sales_count(self, cr, uid, ids, field_name, arg, context=None):
        SaleOrderLine = self.pool['sale.order.line']
        return {
            product_id: SaleOrderLine.search_count(cr,uid, [('product_id', '=', product_id)], context=context)
            for product_id in ids
        }

The result is a mapping from product ids to the number of sale order lines with that product.

Mutable default value
=====================
Never assign to argument mutable default value (context={})::

    def pc(d={}):
        d['count'] = 'count' in d and d['count'] + 1 or 1
        print d['count']

Every time you call pc() function counter will increment.

Useful functions
================

* yield
* zip() and izip()
* sorted()
* reverse()
* iter()
* items() and iteritems()
* heapq()
* collections.namedtuple()
* collections.deque()
* collections.defaultdict()
* <string>.startswith('<first letters>')
* functools.partial()

Substitute this::

    d = {}
    d.setdefault(key, []).append(value)

with this::
    
    d = defaultdict(list)
    d[key].append(value)


else in this case means "no break"::

    for ...:
        ...
        if smth:
            break
    else:  # no break
        do other things


Caching decorator
=================
::
    
    from functools import wraps
    
    @cache
    def make_smth(arg):
        return arg * arg
        
    def cache(func):
        cache = {}  # map from key to value
        @wraps
        def wrapper(*args):
            key = repr(args)
            if not key in cache:
                cache[key] = f(*args)
            return cache[key]
        return wrapper

.. note:: Without the use of this decorator factory (**wraps**), the name of the example function would have been 'wrapper', and the docstring of the original make_smth() would have been lost.


The fastest one key memoization::

    def memodict(f):
        """ Memoization decorator for a function taking a single argument """
        class memodict(dict):
            def __missing__(self, key):
                ret = self[key] = f(key)
                return ret 
        return memodict().__getitem__


The fastest many key memoization::

    def memoize(f):
        """ Memoization decorator for a function taking one or more arguments. """
        class memodict(dict):
            def __getitem__(self, *key):
                return dict.__getitem__(self, key)

            def __missing__(self, key):
                ret = self[key] = f(*key)
                return ret

        return memodict().__getitem__
        
.. warning:: The last two decorators doesn't work with methods.


Monkeypatching
==============

A MonkeyPatch is a piece of Python code which extends or modifies other code at runtime (typically at startup)::


    from SomeOtherProduct.SomeModule import SomeClass
    
    def speak(self):
        return "ook ook eee eee eee!"
    
    SomeClass.speak = speak
    
    
.. note:: You need to get and keep a reference to the original orm.orm and use that instead of the replaced version.

So this is **correct**::

    import osv
    original_orm = osv.orm
    class orm(original_orm):
        def __init__(self, *args, **kw):
            super(orm, self).__init__(*args, **kw)    
        def fields_get(self, *args, **kw):
            print "my fields get................."
            return super(orm, self).fields_get(*args, **kw)    
    osv.orm.orm = orm
    print "replaced.........................."    
    

This is **wrong** and will causes maximum recursion depth exceed exception::    

    from osv import orm
    import osv
    class orm(orm.orm):
        def __init__(self, *args, **kw):
            super(orm, self).__init__(*args, **kw)    
        def fields_get(self, *args, **kw):
            print "my fields get................."
            return super(orm, self).fields_get(*args, **kw)    
    osv.orm.orm = orm
    print "replaced.........................."

http://stackoverflow.com/questions/3781280/how-can-i-replace-the-class-by-monkey-patching


String formatting
=================

==========   =======     =========   =============================================
NUMBER       FORMAT      OUTPUT      DESCRIPTION
==========   =======     =========   =============================================
3.1415926    {:.2f}      3.14        2 decimal places
3.1415926    {:+.2f}     +3.14       2 decimal places with sign
-1           {:+.2f}     -1.00       2 decimal places with sign
2.71828      {:.0f}      3           No decimal places
5            {:0>2d}     05          Pad number with zeros (left padding, width 2)
5            {:x<4d}     5xxx        Pad number with x’s (right padding, width 4)
10           {:x<4d}     10xx        Pad number with x’s (right padding, width 4)
1000000      {:,}        1,000,000   Number format with comma separator
0.25         {:.2%}      25.00%      Format percentage
1000000000   {:.2e}      1.00e+09    Exponent notation
13           {:10d}      13          Right aligned (default, width 10)
13           {:<10d}     13          Left aligned (width 10)
13           {:^10d}     13          Center aligned (width 10)
==========   =======     =========   =============================================

Two digit length::

    month = "{month:0>2d}".format(month=month)


Simple Server
=============

Do you want to quickly and easily share files from a directory? You can simply do::

    # Python2
    python -m SimpleHTTPServer

    # Python 3
    python3 -m http.server


Profiling a script
==================

You can easily profile a script by running it like this::

    python -m cProfile my_script.py


Reversing a list/string
=======================

You can quickly reverse a list by using::

    >>> a = [1,2,3,4]
    >>> a[::-1]
    [4, 3, 2, 1]

This creates a new reversed list. If you want to reverse a list in place you can do::

    a.reverse()

and the same can be applied to a string as well::

    >>> foo = "yasoob"
    >>> foo[::-1]
    'boosay'


Get the caller's method name in the called method
=================================================

For this purpose we can use **inspect** module::

    import inspect

    def f1():
        f2()

    def f2():
        print 'caller name:', inspect.stack()[1][3]

    f1()

Or even just use sys module::

    import sys
    print sys._getframe().f_back.f_code.co_name

See some more detailed example in :ref:`inspect-label`.


Assert that variable is instance method
=======================================

With *inspect.ismethod*::

    import inspect

    def foo(): pass

    class Test(object):
        def method(self): pass

    print inspect.ismethod(foo) # False
    print inspect.ismethod(Test) # False
    print inspect.ismethod(Test.method) # True
    print inspect.ismethod(Test().method) # True

    print callable(foo) # True
    print callable(Test) # True
    print callable(Test.method) # True
    print callable(Test().method) # True


With *type*::

    import types

    def is_instance_method(obj):
        """Checks if an object is a bound method on an instance."""
        if not isinstance(obj, types.MethodType):
            return False # Not a method
        if obj.im_self is None:
            return False # Method is not bound
        if issubclass(obj.im_class, type) or obj.im_class is types.ClassType:
            return False # Method is a classmethod
        return True
