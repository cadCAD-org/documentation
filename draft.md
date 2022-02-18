# cadCAD 1.0 API draft


## Spaces

Spaces are conceptual primitives in cadCAD; they represent the "shape" or "schema" of a signal.

```python=
plane = Space(name="Cartesian Plane")

plane.dimensions
>>> []
```

Spaces contain a set of attributes called dimensions which are typed. However, a space is empty (zero dimensional) until it is equipped with dimensions.

```python=
plane.append_dimension("x", float)
plane.append_dimension("y", float)

plane.dimensions
>>> ['x', 'y']

plane.x
>>> Dimension

plane.x.dtype
>>> float

Type(plane)
>>> Space
```

Although spaces are only shaped containers for information that are critical building blocks. I can make a new space which is like `plane` but is 3 dimensional

```python=
R3 = plane.copy().append("z", float)

R3.dimensions
>>> ['x', 'y', 'z']
```

in addition to building spaces by appending dimensions we can find subspaces by selecting subsets of dimenions

```python=
R2 = R3.subspace(["x","y"])

R2.dimensions
>>> ["x", "y"]
```

the objects 'R2' and 'plane' represent the same abstract space but they are not the same python object.

```python=
plane == R2
>>> False

is_equivalent(plane, R2)
>>> True
```

we would also like to be able to verify equivalence for subspaces
```python=
is_equivalent(plane, R3.subspace(["x","y"]))
>>> True
```

here is a counter example where the dimensions keys match but their values (the associated Types) do not match
```python=
Z2 = Space(name="integer grid")
Z2.append_dimension("x", int)
Z2.append_dimension("y", int)

is_equivalent(Z2, R2)
>>> False
```

We can also build spaces from dictionaries (similarly to what you see with pandas dataframes)

```python=
args = {"x":int, "y":int, "z":int}
Z3 = Space.from_dict(args, name = "3D grid")
```

Now let us consider some other common spaces which require an addition concept, a constraint. Consider the (closed) unit interval 

```python=
interval = Space(name = "unit interval")
interval.append_dimension("t", float)

def condition(point):
    
    t = point.t
    assert Type(l) = float
    
    if  < 0:
        return False
    elif point.t > 1
        return False
    else:
        return True

interval.append_constraint("is_in_range", condition)
interval.constraints
>>> ["is_in_range"]
```

this forces us to think more serious about points, and how they relate to spaces. Let us create a point and see how it behaves

```python=
value = interval.point(t=0.3)

Type(value)
>>> Point

Type(value.t)
>>> float

value.t
>>> 0.3

value.space == interval
>>> True

value.is_in_range()
>>> True
```

but what if i were to instead to try to create a point that failed the contraint

```python=
value = interval.point(t=1.1)

Type(value)
>>> Point

Type(value.t)
>>> float

value.t
>>> 1.1

value.space == interval
>>> True

value.is_in_range()
>>> False
```

i might further add that another way to specify the interval would be as two seperate simpler constraints. Let's try that approach and create a unit_circle and let's use the `is_feasible` method to check that ALL declared constraints are satisfied. 

```python=
S1 = Space(name = "unit circle")
S1.append_dimension("theta", float)
S1.append_constraint("satisfies_upper_bound", lambda point: point.theta < 2*math.pi)
S1.append_constraint("satisfies_lower_bound", lambda point: point.theta >= 0)

S1.constraints
>>> ["satisfies_upper_bound", "satisfies_lower_bound" ]

a270 = S1.point(theta=3*math.pi/2)

a270.is_feasible()
>>> True
```

without any other equipment, we would simple have an awkward point which is associated to the space `interval` but not satisfying its constraint. Depending on context we might do different things with a problematic point such as this one but the most natural thing would be to provide a projection mapping, that is a method which will take any point regardless of whether the constraints are `True` or `False`, and return a point for which they are `True`. Furthermore, we require that a point for which the constraints are `True` returns itself under the projection map. 

```python=
def reduce_angle(point):
    """
    this is the method for reducing any angle to
    the equivalent angle in the range [0,2*Pi)
    its domain is the space S1 (without the interval constraint)
    its codomain is the space S1 (with the interval constraint)
    """
    space = point.space
    theta = point.theta
    remainder = theta % 2*math.pi
    
    if theta >= 0:
        new_theta = remainder
    else:
        new_theta = 2*math.pi + remainder 
        
    return space.point(theta=new_theta) 

S1.set_projection(reduce_angle)

a450 = S1.point(theta=5*math.pi/2)

a450.is_feasible()
>>> False

a450.projection()
>>> 1.5708... #Pi/2 = 90 degrees

a450.projection().is_feasible()
>>> True

```


commentary: its worth noting that this projection mapping is a special case of a block, so too is the constraint. 
+ constraints are blocks that have domain equal to the space (but need not satisfy the constraints), and codomain `bool`.
+ projections are blocks that have domain equal to the space (but need not satisfy the constraints), and have codomain equal to the space (but MUST satisfy the constraints)
+ the API can work as above because all the missing information about the block being created behind the scenes can be inferred from the space itself.
+ by equipping Spaces with the `projection` and `is_feasible` methods we ensure we can handle spaces with characteristics which must be described in terms of restrictions (as is the case with the unit interval)
+ if there are no constraints (`myspace.constraints == []`) then 
    + `is_feasible` returns `True`
    + `point.projection()` returns `point` (that is to say `projection` is the identity map `f = lambda point: point`)
+ when one appends a projection, in the block that is build behind the scenes we should include an `assert new_point.is_feasible()` before returning the new point. this will ensure we throw an error if the projection is not actually a projection. 

Another important operation is Cartesian product, for spaces, the `*` operator will denote Cartesian product. Let's use the Cartesian product to create some Polar Coordinates
```python=
angle = S1.copy()
angle.set_name("angle")
magnitude = Space("magnitude")
magnitude.append_dimension("r", float)
magnitude.append_constraint("is_nonnegative", lambda point: bool(point.r>=0) )

polar_coords = magnitude * angle 
polar_coords.set_name("Polar Coordinates")

def simplify(point):
    """
    a point in polar coordinates with theta in [0, 2Pi) and r>=0
    can always be created from a theta and r even if they violate these constraints
    a negative r becomes positive by adding Pi (180 degrees) to theta
    a theta outside the range can be mapped into the range by removing factors of 2Pi
    """
    if point.is_feasible:
        return point
    else:        
        r = point.r
        theta = point.theta
        if r < 0:
            theta += math.pi
        
        remainder = theta % 2*math.pi
    
        if theta >= 0:
            theta = remainder
        else:
            theta = 2*math.pi + remainder
    
    space = point.space #the space the input point is from
    
    #return a new point from the same space with these values
    return space.point(r=r, theta=theta)
    
polar_coords.set_projection(simplify)


polar_coords.dimensions
>>>["r","theta"]

polar_coord.constraints
>>> ["satisfies_upper_bound", "satisfies_lower_bound", "is_nonnegative" ]

```

This also brings to mind another validation operation which will be important; establishing and checking whether a point and space match.
```python=
plane = Space.from_dict({"x":float, "y":float})
grid = Space.from_dict({"x":int, "y":int})
other_grid = grid.copy()
other_grid.append_dimension("z", int)

pp11 = plane.point(x=1,y=1)
gp11 = grid.point(x=1,y=1)

gp11 == pp11
>>> False

gp11.is_in(grid)
>>>True

gp11.is_in(plane)
>>> False

gp11.is_in(other_grid)
>>> False

gp11.is_in(other_grid.subspace(["x","y"]))
>>> True
```

we can also look at how points can be used with subspaces; in this case we take a point the point (x,y,z)=(1,1,1) in the 3 dimensional grid and project it onto the 2 dimensional grid.
```python=
ogp111 = other_grid.point(x=1,y=1,z=1)

ogp111.project_onto(grid).equals(gp11)
>> True

ogp111.project_onto(other_grid.subspace(["y","z"])).equals(gp11)
>>> False 
```
Can we overload `==` to be this `equals` method; as it stands i believe `==` will not agree with this logic because the points are not the same but they are not the same 'Point' object in memory, however if we look into those 2 objects we will find they have identical contents.

There is one more core concept that defines a space, that concept is a metric. 
https://en.wikipedia.org/wiki/Metric_(mathematics)
A metric is like a constraint in that it takes two points in the domain and returns a `float`. Whereas a constraint takes one point and must return a `bool`.

```python=
def manhattan_distance(point1,point2):
    """
    for a space made up of simple numerical dimensions
    this metric measures manhattan distance between the points
    """
    dist = 0
    for key in point.space.dimensions:
        dist += math.abs(getattr(point1, key) - getattr(point2, key) )
    
    return dist

def euclidean_distance(point1,point2):
    """
    for a space made up of simple numerical dimensions
    this metric measures euclidean distance between the points
    """
    summand = 0
    for key in point.space.dimensions:
        summand += (getattr(point1, key) - getattr(point2, key) )**2
   
    dist = math.sqrt(summand)        
    return dist

grid.append_metric("manhattan", manhattan_distance)
grid.append_metric("euclidean", euclidean_distance)

grid.metrics
>>> ["manhattan", "euclidean"]

gp11 = grid.point(x=1,y=1)
gp45 = grid.point(x=4,y=5)

# |1-4| + |1-5| + 3+4 = 7
grid.manhattan(gp00,gp45)
>>> 7.0 

# sqrt( (1-4)**2 + (1-5)**2) = sqrt(9+16) = 5
grid.euclidean(gp00,gp45)
>>> 5.0 
```
common numerical types will have common distances -- the power of this feature comes in when we start constucting complex spaces from user defined classes and we need to create bespoke metrics for those spaces.

it may also be relevant to give a space an `origin` point, this point will serve as a default or empty value. it is especially useful for measuring other points against under metrics
```python=
grid.set_origin(x=0,y=0)
gp00 = grid.point(x=0,y=0)
gp12 = grid.point(x=1,y=2)

#assuming we've overloaded `==`
grid.origin == gp00
>>> True

grid.manhattan(gp12,grid.origin)
>>> 3
```

in order to truly understand the generality of these spaces, let's construct something all together different. I made a space of 3 word strings but limited those words to contain ascii letters. 

```python=
from string import ascii_letters
from enchant.utils import levenshtein

wordspace3D = Space({"w":str,"u":str, "v":str}, name="space of 3 strings limited to lowecase letters")

#set origin to be the triple of empty strings
wordspace3D.set_origin(w="",u="",v="")

def is_ascii(point):
    """
    check to see if all the 
    """
    all_letters = point.w+point.u+point.v
    bit = True
    for l in len(all_letters):
        if all_letters[l] in ascii_letters:
            pass
        else:
            bit = False
            break
    
    return bit

wordspace3D.append_constraint("is_ascii", is_ascii)

def cleanup(point):
    """
    turn capital letters into lowercase letters
    remove all other characters
    """
    w = point.w
    w_aslist = [l for l in list(w) if l in ascii_letters ]
    w = str(w_aslist)
    
    u = point.u
    u_aslist = [l for l in list(u) if l in ascii_letters ]
    u = str(u_aslist)
    
    v = point.v
    v_aslist = [l for l in list(v) if l in ascii_letters ]
    v = str(v_aslist)
    
    space = point.space
    return space.point(w=w,u=u,v=v)

wordspace3D.set_projection(cleanup)

def edit_distance(point1, point2):
    """
    computes the edit distance (aka levenshtein distance) for each word
    and adds them up
    """
    dist = levenshtein(point1.w, point2.w) + levenshtein(point1.u, point2.u) + levenshtein(point1.v, point2.v)

    return float(dist)
    
wordspace3D.append_metric("levenshtein", edit_distance)

```
A different but related space would be using dictionary of words used for seed phrases, you might say a seedphrase belongs to a space of 24 words that must be drawn from that list. (that might make a better example but i roughed this one out on the fly to make a point about non-numerical spaces still having constrains and metrics). i wouldn't use the example above without first porting it to code and debugging it.

---
:warning: Todo
these patterns work great when the Types are simple, but what happens when we introduce Classes with nonstandard instantiation patters, such as numpy.ndarray.

```python=

```