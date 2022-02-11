API
===

Dimensions
----------

.. code-block:: python

   class Dimension:
      def __init__(self, name, dtype):
         ...

.. code-block:: python

   a = Dimension("a", str)
   b = Dimension("b", int)

   # Note: Types that we wish to impose shape on must be (sub)classed to
   # enforce these restrictions. An example of subclassing numpy.ndarray
   # is given below.
   
   class TwoByTwoInt(np.ndarray):
      def __new__(cls):
         return super().__new__(cls, (2, 2), int)

   # Once a subclass exists that imposes our shape and/or subtypes:

   c = Dimension("c", TwoByTwoInt)

.. autosummary::
   :toctree: generated

   cadcad