# Intervals

`Interval` is a closed interval `[a, b]` with double precision endpoints and directed rounding.
Every operation returns an interval guaranteed to contain all results of the operation applied to points of the
operands.

## Construction

```python
Interval()                  # [0, 0]
Interval(2.5)               # the point interval [2.5, 2.5]
Interval(-1, 1)             # [-1, 1]
Interval("0.1", "0.2")      # endpoints parsed from strings with outward rounding
Interval(x)                 # copy
Interval.pi()               # enclosure of pi
Interval.euler()            # enclosure of e
```

Floats (and ints) are implicitly converted to `Interval`, so `Interval(1) / 3.0` and `2.0 * x` work.

## Arithmetic

| Expression | Meaning |
|------------|---------|
| `x + y`, `x - y`, `x * y`, `x / y`, `-x` | arithmetic with floats or intervals on either side |
| `x += y`, `-=`, `*=`, `/=` | in-place versions |
| `x ** n` | power; `n` may be `int`, `float` or `Interval` |
| `abs(x)` / `x.abs()` | interval of absolute values |

## Comparisons

`==`, `!=`, `<`, `<=`, `>`, `>=` are defined between intervals and between an interval and a float. They
follow the CAPD C++ semantics (a *certified* comparison between sets of values) rather than the semantics of
comparing two floats, so `not x >= 0.1` is **not** the same as `x < 0.1`. See the CAPD documentation of
`interval` for the exact definitions.

## Endpoints, midpoint, set relations

Methods and free functions with the same name are both available:

```python
x = Interval(1, 3)
x.leftBound(), x.rightBound()   # 1.0, 3.0 (floats); inf()/sup() are aliases
x.left(), x.right()             # point intervals containing the endpoints
x.mid()                         # a float near the middle
x.contains(2.0)                 # point membership
x.contains(Interval(1.5, 2))    # interval inclusion
x.containsInInterior(...)       # strict versions
x.subset(y), x.subsetInterior(y)
diam(x), width(x)               # diameter / width
split(x)                        # (mid, remainder) with x ⊂ mid + remainder
intersection(x, y)              # Interval, or None when empty
intervalHull(x, y)              # smallest interval containing both
```

Other helpers: `isSingular`, `isinf`, `isnan`, `copysign`, `nonnegativePart`, `ball(center, radius)`,
`solveAffineInclusion(a, p, c)` (solves `a + [0,t]*p ⊂ c` for `t`), and
`solveAffineInclusionWithDir` which also returns the direction.

## Elementary functions

Available as module-level functions taking and returning `Interval`:

`sqrt`, `sqr`, `power`, `exp`, `log`, `sin`, `cos`, `tan`, `cot`, `asin`, `acos`, `atan`, `atan2`,
`sinh`, `cosh`, `tanh`, `coth`.

```python
a = Interval(1.9058939019661396)
T = 2.0 * Interval(1) / (Interval(0.5) * sin(a))
```

## Typical proof idioms

```python
# Is the enclosure contained in the open set we need?
subset(z[0], s)               # z[0] ⊂ s
(v[0] - u[0]).subset(s)       # method form

# Rigorous bound on a number
e = Interval(-1, 1) * 1e-10   # symmetric neighbourhood of 0
```

## Exceptions

`IntervalError` is raised by invalid interval operations (for example division by an interval containing 0).
