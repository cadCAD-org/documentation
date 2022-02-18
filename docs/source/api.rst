===
API
===

**********
Dimensions
**********

Dimension()
===========

To instantiate a basic Dimension:

.. code-block:: python

   a = Dimension(str, "a")

   # Optionally, we can specify a description
   b = Dimension(int, "b", "my basic dimension")

For complex types where additional restrictions may need to be
considered, we can utilize subclassing:

.. code-block:: python
   
   class TwoByTwoInt64(numpy.ndarray):
      def __new__(cls):
         return super().__new__(cls, (2, 2), numpy.int64)

   # Once a subclass exists that imposes our shape and/or subtypes:
   c = Dimension(TwoByTwoInt64, "c", "my special dimension")

Dimension.copy()
================

By default, Dimensions are immutable. We can copy a Dimension which
results in a new Dimension being instantiated with the same properties
as the source.

.. code-block:: python

   a = Dimension(int, "a")
   a2 = a.copy()

*Note:* Python allows us to shadow variables which enables things like
`a = a.copy()`.

When we copy a Dimension without specifying any arguments all source
properties are copied into the newly instantiated copy. This can cause
problems when adding a Dimension and the respective copies to the same
Space. To address this, you can specify a new name at the time of copy:

.. code-block:: python

   a = Dimension(int, "a")
   a2 = a.copy("a2")

   print(a.name, a2.name) # 'a', 'a2'