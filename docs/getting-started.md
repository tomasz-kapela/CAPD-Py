# Getting started

## Requirements

- Python 3.9 or newer (`pyproject.toml` declares `>=3.8`, the README documents 3.9 as the minimum)
- A C++ compiler, CMake and the Python development headers (`python3-dev`), because the package is built
  from source when installed

## Installation

```bash
pip install uj-capdpy
```

### Building from source

Clone with submodules (CAPD and pybind11 are vendored under `lib/`):

```bash
git clone --recurse-submodules https://github.com/tomasz-kapela/CAPD-Py
cd CAPD-Py
cmake -B build
cmake --build build
```

The build produces a shared object whose name starts with `capdpy` in `build/`. Put it on your `PYTHONPATH`
(or next to your script) and `import capdpy` works.

## A first computation

Compute the image of a point under the flow of the Rössler system after time 2:

```python
from capdpy import *

# Vector field given as a string: parameters, variables, and the right-hand side
vf = IMap("par:a,b;var:x,y,z;fun:-(y+z),x+b*y,b+z*(x-a);")
vf.setParameter("a", Interval(57.) / Interval(10.))
vf.setParameter("b", Interval(2.) / Interval(10.))

solver = IOdeSolver(vf, 20)       # Taylor method of order 20
tm = ITimeMap(solver)             # transport of sets over a time interval

s = C0Rect2Set(IVector([1.0, 1.0, 1.0]))   # a set to transport (here a point)
print(tm(Interval(2.0), s))                # rigorous enclosure at time 2
```

The pattern is the same everywhere:

1. describe the dynamical system with an `IMap`,
2. choose an integrator (`IOdeSolver` for the flow itself or its first derivative, `IC2OdeSolver` for second
   derivatives, `ICnOdeSolver` for arbitrary order),
3. wrap it in a `ITimeMap` or a `IPoincareMap`,
4. evaluate it on a **set** (`C0Rect2Set`, `C1Rect2Set`, ...) chosen to match the derivative order you need.

## Conventions

**Everything is an interval.** Python floats and ints are implicitly converted to `Interval` wherever an
interval is expected, but the conversion is exact only for values representable as doubles. `Interval(0.1)` is
*not* an enclosure of 1/10; write `Interval(1) / Interval(10)` instead (or
`Interval("0.1", "0.1")` using the two-string constructor, which parses the endpoints with outward rounding).

**Output arguments are modified in place.** Many CAPD routines return additional results through their
arguments (derivative matrices, return times, ...). In Python create the object first and pass it:

```python
dF = IMatrix(3, 3)                  # will receive the derivative of the flow
v = poincare_map(s1, dF)            # returns the image, fills dF
```

**Sets are updated in place.** `tm(T, s)` returns the enclosure of the image *and* advances the set `s` by
time `T`. Create a fresh set to repeat a computation.

**Exceptions.** Failures of the underlying library surface as Python exceptions, e.g. `IntervalError` and
`EmptyIntersectionError`.
