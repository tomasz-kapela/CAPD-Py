# Differential algebra: jets, curves, solution curves

This module contains the data structures used to store Taylor expansions. Most users only need
[`ISolutionCurve`](#solution-curves); the rest is useful for higher-order computations.

## IHessian

Second derivatives of a function `R^D → R^M`. Indexed by `(component, i, j)`:

```python
H = IHessian(M, D)
H[0, 1, 2]              # second derivative of component 0 w.r.t. variables 1 and 2
H[0, 1, 2] = Interval(3)
H.dimension(), H.imageDimension()
```

Supports `+`, `+=`, scalar `*`, `*= Interval`, products with an `IMatrix`, `==`, `!=`. Can be built from a
C2 set: `IHessian(s2)`.

## IJet and ICnContainer

`IJet(M, N, degree)` holds the Taylor coefficients (truncated power series) of a map `R^N → R^M` up to the
given degree. `ICnContainer` is its base storage class.

```python
jet = IJet(2, 2, 3)
jet[Multipointer([0, 1])]          # vector of coefficients for ∂x0∂x1
jet()                              # value (vector)
jet(x)                             # evaluate the polynomial at x
jet.derivative(x)                  # derivative of the polynomial at x
print(jet.toString())              # human readable listing
```

Jets support `+`, `-`, `@` (composition product as defined by CAPD), `==`, `!=`. Functions:
`substitutionPowerSeries(first, second)`, `inverseSeriesCloseToIdentity(jet)`, `inversePowerSeries(jet)`.

`ICnContainer` is indexed by `(i, multipointer)`, `(i, multiindex)` or a tuple `(i, a, b, ...)` of variable
indices; `degree()`, `dimension()`, `imageDimension()`, `first()`, `hasNext(...)` iterate coefficients.

## Curves

Classes storing Taylor expansions of a solution on one time step (including dependence on the initial
condition). They are normally obtained from a solver (`solver.getCurve()`), not constructed directly.

| Class | Stores |
|-------|--------|
| `IParametricCurve` | abstract curve; `setDomain`, `getLeftDomain`, `getRightDomain` |
| `BasicCurve`, `IBasicC2Curve`, `IBasicCnCurve` | raw coefficient storage: `setOrder`, `getOrder`, `coefficient(...)`, `remainderCoefficient(...)`, `centerCoefficient(...)`, iterators |
| `ICurve`, `ICurve_C2`, `ICurve_Cn` | evaluation: `curve(t)`, `derivative(t)`, `timeDerivative(t)`, `valueAtCenter(t)`, `remainder(t)`, `getCenter()` |
| `IC2Curve`, `ICnCurve` | additional `hessian(t)`, `setInitHessian`, and for Cn also `jet(t)`, `eval(t)`, `setInitJet` |

## Solution curves

A `ISolutionCurve` stores a whole trajectory as a sequence of curve pieces and is filled by a time map.

```python
solution = ISolutionCurve(0.0)             # left end of the time domain
tm(T, s, solution)

solution(7.)                               # IVector enclosure of x(7)
solution(Interval(3, 4))                   # enclosure of x([3, 4])
solution.derivative(t)                     # derivative w.r.t. initial condition
solution.timeDerivative(t)                 # dx/dt
solution.getLeftDomain(), solution.getRightDomain()
solution.getNumberOfPieces()
```

`ISolutionCurve_C2` and `ISolutionCurve_Cn` are the variants used with `IC2TimeMap` and `ICnTimeMap`.
All solution curve classes expose `hessian(t)` and `jet(t)` (meaningful when the solver computed that order).

Typical use — verifying a constraint along the whole trajectory (from
`examples/check-collisions.py`):

```python
N = 200
for i in range(N):
    t = Interval(i, i + 1) * T / N
    if not solution(t).euclNorm() >= 0.1:
        print("Constraint not satisfied for t =", t)
        break
```

## Time jets

`ITimeRange` (stores current time), `IC0TimeJet`, `IC1TimeJet`, `IC2TimeJet`, `ICnTimeJet` bundle a set's
value, derivative, Hessian or jet together with its current time. They are the low-level argument type of the
solver `__call__` overloads; normally you work with the set classes instead.
