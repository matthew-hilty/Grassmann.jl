<p align="center">
  <img src="./docs/src/assets/logo.png" alt="Grassmann.jl"/>
</p>

# Grassmann.jl

*Static dual multivector types with graded-blade indexing, product algebra, and optional origin*

[![Build Status](https://travis-ci.org/chakravala/Grassmann.jl.svg?branch=master)](https://travis-ci.org/chakravala/Grassmann.jl)
[![Build status](https://ci.appveyor.com/api/projects/status/c36u0rgtm2rjcquk?svg=true)](https://ci.appveyor.com/project/chakravala/grassmann-jl)
[![Coverage Status](https://coveralls.io/repos/chakravala/Grassmann.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/chakravala/Grassmann.jl?branch=master)
[![codecov.io](http://codecov.io/github/chakravala/Grassmann.jl/coverage.svg?branch=master)](http://codecov.io/github/chakravala/Grassmann.jl?branch=master)

This package is a work in progress providing the necessary tools to work with arbitrary dual multivectors with optional origin. Full multivector types are not utilized unless necessary. For example, an operation on two blades may result in another blade type instead of a full multivector type. Both static and mutable vector types are supported.

### Requirements

This requires my forked version of `ComputedFieldTypes` at https://github.com/chakravala/ComputedFieldTypes.jl

## Usage

Let `N` be the dimension of the algebra. The metric `Signature{N}` of the basis can be specified with the `S"..."` constructor by using `+` and `-` to specify whether the basis element of the corresponding index squares to `+1` or `-1`.
For example, `S"+++"` constructs the signature of a positive definite 3-dimensional vector space.
Optionally, the final basis element can be specified as the origin by using `o` instead, i.e. `S"+++o"`.
This basis element will be treated in a special way, such that its meaning will be retained for the sake of combining algebras of different dimensions when the `Signature` is compatible.
Likewise, to get dual numbers, it is also possible to specify a degenerate basis element with `ϵ` (having property `ϵ*ϵ=0`) at the first index, i.e. `S"ϵ+++"`.
The dimension of the algebra corresponds to the total number of basis vectors, but even though `S"ϵ+++o"` corresponds to a `5`-dimensional algebra it can be internally recognized as being generated by a `3` dimensional vector space due to the encoding of the degenerate basis element and origin in the `Signature` type.
With julia's multiple dispatch, methods can easily specialize on the algebra's types.

The elements of the algebra can currently be generated using the basis elements created by the `@basis` macro,
```Julia
julia> using Grassmann; @basis e s "ϵ+++"
(ϵ+++, e, eϵ, e₁, e₂, e₃, eϵ₁, eϵ₂, eϵ₃, e₁₂, e₁₃, e₂₃, eϵ₁₂, eϵ₁₃, eϵ₂₃, e₁₂₃, eϵ₁₂₃)
```
As a result of this macro, all of the basis elements with that signature become available in the local workspace with the specified naming.
The first argument provides the prefix of the basis names, the second argument is the variable name for the signature, and the third argument is the signature. Thus,
```Julia
julia> s
ϵ+++

julia> typeof(s)
Signature{4}

julia> typeof(e13)
Basis{4,2}

julia> e13*e2
-1e₁₂₃

julia> ans * eϵ
1eϵ₁₂₃

julia> ans * eϵ
0e
```
It is entirely possible to assign multiple different bases with different signatures without any problems.
```Julia
julia> @basis b S "++++"
       let k = (b1+b2)-b3
           for j ∈ 1:9
               k = k*(b234+b134)
               println(k)
       end end
0 + 1e₁₄ + 1e₂₄ + 2e₃₄
0 - 2e₁ - 2e₂ + 2e₃
0 - 2e₁₄ - 2e₂₄ - 4e₃₄
0 + 4e₁ + 4e₂ - 4e₃
0 + 4e₁₄ + 4e₂₄ + 8e₃₄
0 - 8e₁ - 8e₂ + 8e₃
0 - 8e₁₄ - 8e₂₄ - 16e₃₄
0 + 16e₁ + 16e₂ - 16e₃
0 + 16e₁₄ + 16e₂₄ + 32e₃₄
```
Alternatively, if you do not wish to assign these variables to your local workspace, the `Grassmann.Algebra{N}` constructors can be used to contain them.
```Julia
julia> G3 = Grassmann.Algebra(S"+++")
Grassmann.Algebra{3,8}(+++, e, e₁, e₂, e₃, e₁₂, e₁₃, e₂₃, e₁₂₃)

julia> G3.e13 * G3.e12
e₂₃
```
This package is still a work in progress, and the API and implementation may change as more features and algebraic operations and product structure are added.
