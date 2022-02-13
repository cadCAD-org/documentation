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

   #it is possible to create copies of dimensions
   a2 = a.copy()

   print(a2.name)
   # >>> "a"

   #in order to avoid namespace collisions it is prudent to give copies new names
   a2 = a.copy(name="a2")
   print(a2.name)
   # >>> "a2"

Spaces
------

.. code-block:: python

   class Space:
      def __init__(self, dimensions, names = None, override=False):
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
   s = Space((x, y, z, Z), names = ["x", "y", "z", "zed"])

   # alternatively, we can first create a Space without collisions and
   # then append a single dimension with an explict name
   s = Space((x, y, z))
   s.append_dim(Z, name="zed") # a new space object with the additional dimension now called s

   # for maximally explicit declaration of a new space, it can be created
   # from a dictionary
   space_dict = {"x":x, "y":y, "z":z, "zed":Z}
   s = Space.from_dict(space_dict)

   print(s.dimensions.keys()) # ('x', 'y', 'z', 'zed')

   # Consider the case of many collisions

   a = Dimension(int, "a")
   b = Dimension(int, "a")
   c = Dimension(int, "a")
   d = Dimension(int, "a")

   s = Space((a, b, c, d)) # Throws exception

   # Fixed with...
   s = Space((a, b, c, d), names = ["a", "my_a", "my_other_a", "my_other_other_a"])

   # or by using the from_dict method
   space_dict = {"a":a, "my_a":b, "my_other_a":c, "my_other_other_a":d}
   s = Space.from_dict(space_dict)

   print(s.dimensions.keys()) 
   # >>> ('a', 'my_a', 'my_other_a', 'my_other_other_a')

   print([d.name for d in s.dimensions])
   # >>> ('a', 'a', 'a', 'a')

   # it may be clear that having a bunch of dimensions named "a"
   # could create a lot of confusion, even if they are different
   # objects in memory
   print(s.my_other_a.name)
   # >>> "a"

   #however if we flip the override bit to True
   # then new renamed copies of those dimensions are used
   s = Space((a, b, c, d), names = ["a", "my_a", "my_other_a", "my_other_other_a"], override = True)
   print(s.my_other_a.name)
   # >>> "my_other_a"

   #the same behavoir holds for from_dict
   space_dict = {"a":a, "my_a":b, "my_other_a":c, "my_other_other_a":d}
   s = Space.from_dict(space_dict, override=True)

   print(s.my_other_a.name)
   # >>> "my_other_a"

.. autosummary::
   :toctree: generated

   cadcad