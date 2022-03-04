===============
API Exploration
===============

******
Spaces
******

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
   s = Space((a, b, c, d), names = ["a", "my_a", "my_other_a", "my_other_other_a"], override = True)
   print(s.my_other_a.name) # "my_other_a"

   # The same override option is available with the from_dict() method:
   space_dict = {"a": a, "my_a": b, "my_other_a": c, "my_other_other_a": d}
   s = Space.from_dict(space_dict, override = True)

   print(s.my_other_a.name) # "my_other_a"

******
Blocks
******

.. code-block:: python

   # A collection of ages will define our initial state space
   d1 = Dimension(int, "Alice", "Age of Alice")
   d2 = Dimension(int, "Bob", "Age of Bob")
   d3 = Dimension(int, "Carol", "Age of Carol")
   d4 = Dimension(int, "David", "Age of David")

   # And an average of all the above ages will define our updated state space
   d5 = Dimension(float, "Average", "Average age")

   s1 = Space((d1, d2, d3, d4)) # Represents a domain (initial state space)
   s2 = Space((d5)) # Represents a codomain (updated state space)
   
   p1 = Space() # Represents our param space (empty)

   def fn(point, codomain):
      return Point(codomain, {"Average": sum(point.values()) / len(point)})

   # Domain and codomain arguments can be single space objects or collections of space objects
   b1 = Block(fn, s1, s2, p1) # Average age

   init_point = Point(s1, {"Alice": 12, "Bob": 54, "Carol": 76, "David": 25}) # Conforms to s1

   print(init_point.space == s1) # true
   print(init_point.space == s2) # false
   
   next_point = b1.map(init_point)

   print(next_point.space == s2) # true
   print(next_point.space == s1) # false

******
Points
******

.. code-block:: python

   # Note: Instantiation of a point MUST validate that supplied point data
   # satisfies the schema of the supplied space; if the point data does NOT
   # satisfy the space schema, we raise an exception -- otherwise, return
   # the point as per normal

   # Setup a single-dimension space to create points of
   d1 = Dimension(int, "Age", "My Age")
   s1 = Space((d1), "My Space", "My Space is a social network lol")

   # A point is returned as the supplied point data satisfies s1
   p1 = Point(s1, {"Age": 35})
   
   # This would raise an exception since the supplied point data does NOT satisfy s1
   p2 = Point(s1, {"Name": "Tyler"})

   # The space that was supplied at the time of instantiation should be added to the
   # point object
   print(p1.space) # <Space>
   print(p1.space.name) # 'My Space'
   print(p1.space.description) # 'My Space is a social network lol'

   # The reason we add the space to the point itself is so that we can easily check
   # any point against a space for equality
   print(p1.space == s1) # True
   
   s2 = Space(())
   print(p1.space == s2) # False

   # Because our point data must have satisfied the space schema during instantiation
   # we dont need to do any sort of type checks afterwards

   print(p1.space.dimensions.Age.dtype == type(p1.Age)) # No need to do this!

   # Lastly, we should be able to access all data on the point like so
   print(p1.Age) # 35

************
Trajectories
************

   .. code-block:: python

      d1 = Dimension(int, "A")
      d2 = Dimension(str, "S")
      g3 = Dimension(str, "L")

      s1 = Space((d1), "A")
      s2 = Space((d1, d2), "A/S")
      s3 = Space((d1, d2, d3), "A/S/L")

      p1 = Point(s1, {"A": 35})
      p2 = Point(s2, {"A": 35, "S": "Male"})

      def fn(point, codomain):
         return Point(codomain, {"A": point.A, "S": point.S, "L": "Utah"})

      # Create a block that takes an s2 point (we created one called p2 above) and
      # adds a hardcoded location of "Utah" to create a new s3 point which is returned.
      b1 = Block(fn, s2, s3) # fn, domain, codomain

      t1 = Trajectory(p1) # You must initialize with one or more point; at this point t1 can only contain s1 type points
      t2 = Trajectory(p2) # Every point in this trajectory must be of type s2

      print(t1) # (<point>)

      # We can add to our t2 trajectory another s2 point by running a block!
      # p2 is a point that conforms to s2; the blocks .map() runs fn() which
      # returns a new point that conforms to our codomain of s3
      t1.add_point(b1.map(p2))

      print(t1) # (<point>, <point>)