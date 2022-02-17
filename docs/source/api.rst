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

   # To create a new dimension from another, we can use .copy()
   a2 = a.copy()

   # This creates a new Dimension with the same properties (type, name,
   # and description) as the source. To create a new copy that will not
   # cause conflict when being added to the same space as the original,
   # we can specify a new name at the time of copy:

   a3 = a.copy("a3")
   print(a.name, a2.name, a3.name) # "a", "a", "a3"

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
   # second parameter
   s = Space((x, y, z, Z), names = ["x", "y", "z", "zed"])

   # Alternatively, we can first create a space without collisions and
   # then append a single dimension with an explict name. By default,
   # spaces are immutable and a new space will be created for this reason
   # whenever we append a new dimension
   s = Space((x, y, z))
   s2 = s.append(Z, "zed") # A new space

   # Python allows us to reuse variable names so we are able to do:
   s = Space((x, y, z))
   s = s.append(Z, "zed")

   # For maximally explicit declaration of a new space, it can be created
   # from a dictionary:
   space_dict = {"x": x, "y": y, "z": z, "zed": Z}
   s = Space.from_dict(space_dict)

   print(s.dimensions.keys()) # ('x', 'y', 'z', 'zed')

   # Consider the case of many collisions

   a = Dimension(int, "a")
   b = Dimension(int, "a")
   c = Dimension(int, "a")
   d = Dimension(int, "a")

   s = Space((a, b, c, d)) # Throws exception

   # Fixed with...
   s = Space((a, b, c, d), ["a", "my_a", "my_other_a", "my_other_other_a"])

   # Or by using the from_dict method which is passed a predetermined dict
   # without key/name conflicts
   space_dict = {"a": a, "my_a": b, "my_other_a": c, "my_other_other_a": d}
   s = Space.from_dict(space_dict)

   # A dimensions local name (Dimension.name) will remain 'a' while only its
   # key in the dimensions dict of a space will be updated to avoid conflicts
   print([d.name for d in s.dimensions]) # ('a', 'a', 'a', 'a')

   print(s.dimensions.keys()) # ('a', 'my_a', 'my_other_a', 'my_other_other_a')

   # While providing alternative keys/names to our space does avoid conflicts
   # it doesn't always result in clarity. It could be confusing to have many
   # dimensions with the same local name (Dimension.name). To address this, you
   # may override the local name by providing an override boolean a the time of
   # space instantiation:
   s = Space((a, b, c, d), names=["a", "my_a", "my_other_a", "my_other_other_a"], override=True)
   print(s.my_other_a.name) # "my_other_a"

   # The same override option is available with the from_dict() method:
   space_dict = {"a": a, "my_a": b, "my_other_a": c, "my_other_other_a": d}
   s = Space.from_dict(space_dict, override=True)

   print(s.my_other_a.name) # "my_other_a"

.. autosummary::
   :toctree: generated

   cadcad