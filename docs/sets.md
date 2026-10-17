# Sets

Rigorous integration transports *sets*, not just points. A set class stores an enclosure of a subset of Rn in
a form that limits the wrapping effect, and — for the C1, C2, Cn classes — an enclosure of the derivatives of
the flow with respect to the initial condition.

Choose the set class that matches the derivative order you need **and** is supported by the solver (see the
table in [ODE solvers](ode-solvers.md#which-sets-each-solver-accepts)).

| Class | Stores | Typical solver |
|-------|--------|----------------|
| `C0Rect2Set` | enclosure of the set, doubleton representation | `IOdeSolver` |
| `C0TripletonSet` | tripleton representation (better for long integration times and strong contraction/expansion) | `IOdeSolver` |
| `C0HORect2Set`, `C0HOTripletonSet` | as above, using Hermite–Obreshkov intersection | `IOdeSolver` |
| `C1Rect2Set` | set + first derivative (Lohner C¹ algorithm) | `IOdeSolver` |
| `C1HORect2Set` | C1 with Hermite–Obreshkov | `IOdeSolver` |
| `C11Rect2Set` | C1 set where both parts use QR reorganization | `IOdeSolver` |
| `C2Rect2Set` | set + first and second derivatives | `IC2OdeSolver` |
| `CnRect2Set` | set + all derivatives up to a given degree | `ICnOdeSolver` |

The HO classes require a Taylor order between **3 and 64**.

## Construction

All of the classes take the same family of arguments (C0, C1, C2; `CnRect2Set` takes an extra `degree`
before `t`):

```python
C0Rect2Set(x, t=0)                       # x: IVector, a point or box center
C0Rect2Set(x, r0, t=0)                   # x + r0   (r0: IVector of intervals)
C0Rect2Set(x, C, r0, t=0)                # x + C*r0 (C: IMatrix)
C0Rect2Set(x, C, r0, r, t=0)             # x + C*r0 + r
C0Rect2Set(x, C, r0, B, r, t=0)          # x + C*r0 + B*r
```

`t` is the initial time (it matters for non-autonomous systems).

```python
s0 = C0Rect2Set(IVector([x0, y0, px, py]))               # a (thin) box
s1 = C1Rect2Set(IVector([x0, y0, px, py]))               # + derivative, initially the identity
s2 = C2Rect2Set(u0, 2.)                                  # starts at t = 2
sn = CnRect2Set(x, 3)                                    # derivatives up to degree 3
s  = C0TripletonSet(IVector([0., -8.38, 0.0296]))
```

`CnRect2Set` has additional constructors taking an `IJet` as initial condition.

## Reading values out of a set

```python
IVector(s0)           # enclosure of the set                     (any set)
IMatrix(s1)           # enclosure of the derivative d phi/d x    (C1, C2, Cn sets)
IHessian(s2)          # enclosure of second derivatives          (C2 sets)

s2[i]                 # value of component i
s2[i, j]              # d f_i / d x_j
s2[i, j, k]           # second derivative entry (Taylor coefficient form)

sn[i, multiindex]     # Taylor coefficient for a Multiindex/Multipointer
sn[multipointer]      # vector of Taylor coefficients for a multipointer
```

Note: Taylor coefficient form means `s[i,j,k]` is the normalized coefficient `(1/α!) ∂^α f_i`; multiply by the
factorial to get the plain derivative. In the repository example this is why the second derivative is written
`2*s[0, 1, 1]`.

All sets also derive from `TimeRange`, so `getCurrentTime()` / `setCurrentTime(t)` are available.

## Internal representation

The doubleton and tripleton classes expose their parts for advanced use:

- `AffineSet`: `x + B*r` — accessors `get_x`, `get_r`, `get_B`, `get_invB`, `set_*`, `setElement_*`,
  `getRow_B`, `setRow_B`, ...
- `DoubletonSet`: `x + C*r0 + B*r` — adds `get_r0`, `get_C`, ...
- `CenteredTripletonSet`: `x + C*r0 + (B*r ∩ Q*q)`
- `MatrixAffineSet` / `MatrixDoubletonSet`: sets of matrices `D + Bjac*R` and `D + Cjac*R0 + Bjac*R`
  used for the derivative part of C1/C2 sets.

`affineTransformation(M, x0)` on doubleton, tripleton and HO sets computes the image of the set under the affine
map `v → M (v - x0)`.
