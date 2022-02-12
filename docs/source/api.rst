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

   # To resolve this, we can supply a key override dictionary as our
   # second param
   s = Space((x, y, z, Z), {"z": "zed"})

   print(s.dimensions.keys()) # ('x', 'y', 'z', 'zed')

   # We could even support a list of strings in the key override dict
   # to handle more than one name collision

   a = Dimension(int, "a")
   b = Dimension(int, "a")
   c = Dimension(int, "a")
   d = Dimension(int, "a")

   s = Space((a, b, c, d)) # Throws exception

   # Fixed with...
   s = Space((a, b, c, d), {"a": ["my_a", "my_other_a", "my_other_other_a"]})

   print(s.dimensions.keys()) # ('a', 'my_a', 'my_other_a', 'my_other_other_a')

.. autosummary::
   :toctree: generated

   cadcad