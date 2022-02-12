API
===

Dimensions
----------

.. code-block:: python

   class Dimension:
      def __init__(self, dtype, name, description=None):
         ...

.. code-block:: python

   a = Dimension(str, "a")
   b = Dimension(int, "b")

   # Note: Types that we wish to impose shape on must be (sub)classed to
   # enforce these restrictions. An example of subclassing numpy.ndarray
   # is given below.
   
   class TwoByTwoInt64(numpy.ndarray):
      def __new__(cls):
         return super().__new__(cls, (2, 2), numpy.int64)

   # Once a subclass exists that imposes our shape and/or subtypes:

   c = Dimension(TwoByTwoInt64, "c", "my special dimension")

Spaces
------

.. code-block:: python

   class Space:
      def __init__(self, dimensions, overrides={}):
         ...

.. code-block:: python

   x = Dimension(int, "x")
   y = Dimension(int, "y")
   z = Dimension(int, "z")
   Z = Dimension(int, "z") # Note the capitalization of this variable

   # When we try to create a space from our four dimensions, an exception
   # throws due to z.name and Z.name being equal
   s = Space((x, y, z, Z))

   # To resolve this, we can supply an key override dictionary as our
   # second param
   s = Space((x, y, z, Z), {"z": "zed"})

   print(s.dimensions.keys()) # ('x', 'y', 'z', 'zed')

.. autosummary::
   :toctree: generated

   cadcad