Crash Course
============

.. _introduction:

Introduction
------------

Welcome to the cadCAD crash course. The intent of this course is to walk you through the process of modeling a system. We will take you through the planning phase of model construction through the actual implementation of the model via the :ref:`cadCAD` framework.

Before we jump into the fun stuff, let's talk a little bit about what cadCAD is and why it exists.

.. _cadcad:

What is cadCAD?
---------------

Complex Adaptive Dynamics Computer Aided Design (*cadCAD*) is a framework -- often we consider it to be a full-fledged *language* -- for encoding Generalized Dynamical Systems (GDS) as computer programs. Originally built for internal use at `BlockScience<https://block.science>`, the original architecture was designed by Dr. Michael Zargham, Kurtis Koch, and Matt Barlin in 2017. The first implementation was put into code by Joshua Jodesty later that same year. In 2019, the software was named *cadCAD* and was opensourced in collaboration with the `Commons Stack<https://commonsstack.org>`.

.. _systems:

Identifying Systems
-------------------

What exactly are we talking about when we say *systems*? How would we know a system if and when we saw one? In the context of *cadCAD*, we are most often referring to `dynamical systems<https://en.wikipedia.org/wiki/Dynamical_system>`. Dynamical systems are systems in which a point in ambient space and its time dependency can be described through a function. At any given moment in time, the dynamical system has a state which represents a point in its associated state space.

Put more simply, a dynamical system -- through the use of functions -- define the parameters that restrict information coming into the system and information leaving the system.

.. _diagrams:

Diagramming Systems
-------------------

Need to do.

.. _dimensions:

Dimensions
----------

Dimensions are a core concept in dynamical systems and as such are very important in cadCAD. In their most basic form, a dimension is simply a data type which sufficiently restricts a given piece of information to a specific 'shape'. Dimensions are the most atomic-level unit of information with which we work and the composition of one or more dimensions make up our state space.

Some examples of dimensions include: *int*, *string*, *int64* and *float*

However, language-specific primitive types as demonstrated above are not the only permissible dimensions in dynamical systems. We can construct our own arbitrary data types and use those as dimensions to. Classes and Structs are examples of this flexibility.

.. _spaces:

Spaces
------
Spaces, or *state spaces*, represent collections of dimensions that fully describe the information that can flow through a particular system. More specifically they define what the form information must take when coming into the system as well as the form it must take when leaving. This purpose will become more obvious as we explore :ref:`blocks`.

When describing information as it exists at arbitrary :ref:`points` in time, we can understand exactly what that information must look like by referencing its associated space. More on this later.

.. _blocks:

Blocks
------

Blocks are the last of our core concepts in cadCAD and are where all the fun stuff happens. Blocks are composable structures that define an input space, an output space, and a function for translation of information between the two.

Note: Technically, there are other things that can go into a block but we will explore those at a later time after we've had a chance to build out a strong understanding of the fundamentals.

Blocks are simple systems - albeit perhaps not very useful ones. A block allows us to define information (a signal) entering the system, the information as it leaves the system, and the processes that transform that information from one "shape" to another. Most useful systems are systems that include many blocks strung together with control mechanisms to implement branching, looping, etc. Furthermore, blocks can be composed of other blocks or systems of blocks. We call these types of constructs, :ref:`dynamics`.

The input spaces to a block are generally referred to as the blocks *domain* while the output spaces are referred to as the blocks *codomain*.

.. _points:

Points
------

Perhaps the last crucial concept to master in cadCAD would be that of points. Points are instances of information (state) that satisfy the 'shape' requirements of their respective space. Let's look at a super contrived example of points in play:

Let's describe, using the concepts that we've learned about thus far, a system that takes a collection of ages and outputs the average age. In this example, we will also say that the collection of ages comes from three persons only: *alice*, *bob*, and *charlie*.

For our dimensions, we need only two to be defined: *int* (describes our individual ages) and *float* (describes our average age)

Our single block system will need an input space (domain) defining the data coming into the block as well as an output space (codomain). Our first space could be represented as such (expressed in JSON for demonstration purposes):

.. code-block:: JSON

  {
    "aliceAge": int,
    "bobAge": int,
    "charlieAge": int
  }


The codomain could be represented with:

.. code-block:: JSON

  {
    "averageAge": float
  }

And perhaps our block function could be (in Python and written for clarity of concept):

.. code-block:: Python

  def calculate_average(point): 
    average = (point["aliceAge"] + point["bobAge"] + point["charlieAge"]) / 3
    return {"averageAge": average}

In order for our block to do something, we must first create a starting point which can serve as our initial input to the block:

.. code-block:: JSON

  {
    "aliceAge": 10,
    "bobAge": 20,
    "charlieAge": 30
  }

If we were 'execute' our block by passing in the above point, our block function *calculateAverage* would return a new point with our calculated average age.

.. code-block:: JSON

  {
    "averageAge": 20.0
  }

A few things must be noted here: first, the point we fed our block HAD to satisfy the shape/restrictions imposed by our domain space. If you recall, our space defined three pieces of data all of which were of type *int*. The point we constructed as our initial point does indeed satisfy this space, so the block was able to complete the transformation of the point into a new point which itself satisfies the requirements imposed by our codomain space.

.. _dynamics:

Dynamics
--------

.. _trajectories:

Trajectories
------------

Resources
---------
- [cadCAD Formal Specification](https://raw.githubusercontent.com/cadCAD-org/cadcad-ri/master/docs/formal_specification.pdf)