# SetProgramming

**This package is still a sketch, no code has been written yet. The content of the README shows the goal, not what already works.**

JuMP extension for Set Programming : optimization with set variables and inclusion/containment constraints. This package allows the formulation of a mathematical programming involving both classical variables and constraints supported by JuMP and set variables and constraints.

Two options exist to solve Set Programming:
* [Polyhedral Computation](https://github.com/JuliaPolyhedra/Polyhedra.jl).
* Automatically reformulation into a semidefinite program using [Sum-Of-Squares Programming](https://github.com/JuliaOpt/SumOfSquares.jl) and the S-procedure.

## Variables

The variables can either be
* a polyhedron;
* an Ellipsoid or more generally the 1-sublevel set of a polynomial of degree `2d`;
* a quadratic cone or more generally the 0-sublevel set of a polynomial of degree `2d`

```julia
@variable m S Polyhedron()
@variable m S Ellipsoid()
@variable m S PolySet(d) # 1-sublevel set of a polynomial of degree 2d
@variable m S PolySet(d, convex=true) # Convex 1-sublevel set of a polynomial of degree 2d
@variable m S PolySet(d, symmetric=true) # 1-sublevel set of a polynomial of degree 2d symmetric around the origin
@variable m S PolySet(d, symmetric=true, center=[1, 0]) # 1-sublevel set of a polynomial of degree 2d symmetric around the [1, 0]
@variable m S QuadCone()  # Quadratic cone
@variable m S PolyCone(d) # 0-sublevel set of a polynomial of degree 2d
```

## Expressions

The following operations are allowed

| S + x    | Translation of `S` by `x`     |
| S1 + S2  | Minkowski sum                 |
| S1 ∩ S2  | Intersection of `S1` and `S2` |
| S1 ∪ S2  | Union of `S1` and `S2`        |
| A*S      | Linear mapping                |
| polar(S) | Polar of S                    |

## Constraints

The following constraints are implemented

| x ∈ S    | `x` is contained in `S`  |
| S1 ⊆ S2  | `S1` is included in `S2` |
| S1 ⊇ S2  | `S1` is included in `S2` |
| S1 == S2 | `S1` is equal to `S2`    |

## Examples

Consider a polytope
```julia
using Polyhedra
P = @set x + y <= 1 && x >= 0 && y >= 0
```
Pick an SDP solver (see [here](juliaopt.org) for a list)
```julia
using CSDP # Optimizer
optimizer = CSDPOptimizer()
```

To compute the maximal ellipsoid contained in a polytope (i.e. [Löwner-John ellipsoid](https://github.com/rdeits/LoewnerJohnEllipsoids.jl))
```julia
using JuMP
m = Model(optimizer=optimizer)
@variable m S Ellipsoid()
@constraint S ⊆ P
@objective Max vol(S)
solve()
```

To compute the maximal invariant set contained in a polytope
```julia
using JuMP
m = Model(optimizer=optimizer)
@variable m S Polyhedron()
@constraint S ⊆ P
@constraint A*S ⊆ S # Invariance constraint
@objective Max vol(S)
solve()
```

To compute the maximal invariant ellipsoid contained in a polytope
```julia
using JuMP
m = Model(optimizer=optimizer)
@variable m S Ellipsoid()
@constraint S ⊆ P
@constraint A*S ⊆ S # Invariance constraint
@objective Max vol(S)
solve()
```

To compute the maximal algebraic-invariant ellipsoid (i.e. `AS ⊆ ES`) contained in a polytope:
```julia
using JuMP
m = Model(optimizer=optimizer)
@variable m S Ellipsoid()
@constraint S ⊆ P
@constraint A*S ⊆ E*S # Invariance constraint
@objective Max vol(S)
solve()
```
