# Maps and functions

## IMap

`IMap` represents a map `R^n → R^m` (typically a vector field or a discrete dynamical system) given by a
formula string. It can evaluate itself and compute derivatives automatically.

```python
IMap(formula, degree=1)     # degree is positional-only
```

`degree` is the maximal order of space-derivatives the map will be able to compute (use `2` for second
derivatives, higher for jets).

### Formula syntax

```
par:<parameters>;var:<variables>;fun:<components>;
par:<parameters>;time:<time variable>;var:<variables>;fun:<components>;
```

- `par:` comma-separated parameter names (optional)
- `time:` name of the independent variable for non-autonomous systems (optional)
- `var:` comma-separated state variables
- `fun:` comma-separated right-hand sides
- `^` is power, so `x^2` is x²

```python
f   = IMap("par:c;var:x,y;fun:y,c*y*(1-x^2)-x;")                     # van der Pol
rhs = IMap("par:pi,L,v;time:t;var:x,a;fun:cos((0.5*pi/L)*(t*(v*sin(a))-L))+v*cos(a),0;", 2)
```

Standard functions such as `sin`, `cos`, `exp`, `log`, `sqrt` can be used in expressions; see the CAPD
documentation of the map parser for the full grammar.

### Parameters and time

```python
f.setParameter("c", Interval(2) / Interval(10))   # by name
f.setParameter(0, Interval(0.2))                  # by index
f.setParameters([Interval(1), Interval(2)])       # all parameters, in order
f.getParameter("c")
f.setCurrentTime(Interval(0.0))                   # value of the time variable
f.getCurrentTime()
```

### Dimensions

`f.dimension()` is the number of variables (time and parameters are not counted); `f.imageDimension()` is
the number of components; `f.degree()` / `f.setDegree(d)` is the maximal derivative order.

### Evaluation

```python
u = IVector([1.0, 0.0])

f(u)                    # value                       -> IVector
f(t, u)                 # value at time t             -> IVector
f(u, D)                 # value + derivative          D: IMatrix (output)
f(t, u, D)
f(u, D, H)              # value, derivative, hessian  H: IHessian (output)
f(t, u, D, H)
f.derivative(u)         # derivative only             -> IMatrix
f.derivative(t, u)
f(jet)                  # propagate a jet (IJet) through the map
f(u, jet)               # jet at u (output)
```

### Taylor coefficients of ODE solutions

For an ODE `x' = f(x)` the Taylor coefficients of the solution are computed with:

```python
coeffs = f.computeODECoefficients(u0, order)                    # list of order+1 IVectors
coeffs, dcoeffs = f.computeODECoefficients(u0, dU0, order)       # + variational equation
coeffs, dcoeffs, hcoeffs = f.computeODECoefficients(u0, dU0, h0, order)
jets = f.computeODECoefficients(jet0, degree, order)             # all derivatives up to degree
```

`coeffs[k]` is the k-th normalized Taylor coefficient `x⁽ᵏ⁾(0)/k!`.

```python
vfield.setCurrentTime(0.0)
coeffs = vfield.computeODECoefficients(u0, 2)
for i in range(3):
    print("Taylor coeffs[", i, "] =", coeffs[i])
```

For a non-autonomous system call `setCurrentTime` first.

### Masks

`setMask(list_of_multiindices)`, `addMultiindexToMask(mi)`, `resetMask()` and `getMask(...)` restrict
derivative computations to a chosen set of partial derivatives (CAPD adds all dependencies automatically).

## IFunction

`IFunction` is the scalar-valued analogue, convenient for sections and constraints.

```python
g = IFunction("par:a;var:x,y;fun:x^2+y^2-a;")
g.setParameter("a", Interval(1))
g(IVector([0.6, 0.8]))     # value at a vector
g.gradient(IVector([0.6, 0.8]))
```

For a function of one variable `g(Interval)` is also accepted.

`IMap` and `IFunction` share the parameter and time methods of their common base class `IBasicFunction`.
