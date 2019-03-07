---
layout: post
title: Finding the Check Axis
---
For my
[cut and project tiling generator](https://github.com/gglouser/cut-and-project-tiling),
I needed to test whether a point was in a region bounded by
pairs of parallel hyperplanes. In the
[small overview](https://gglouser.github.io/cut-and-project-tiling/docs/intro.html),
I described what I called a *check axis*:

> We need to test if a point falls inside the projection, or
> *shadow*, of a hypercube in the dual space. The hypercube
> shadow forms a region bounded by pairs of parallel hyperplanes
> in the dual space. A point must be between every pair of
> bounding hyperplanes to be inside the shadow.

> Now we have reduced the problem to testing whether a point
> falls between a pair of parallel hyperplanes. Represent each
> hyperplane pair by the unique vector (in the dual space)
> orthogonal to them, which I will refer to as a *check axis*.

Here is how I came up with formula for the check axes.
It uses the language of geometric algebra.

---

We have an $$n$$-dimensional square lattice aligned with elementary basis vectors
$$\{ e_1 , \ldots , e_n \}$$. We also have a cutting plane $$B$$,
with orthonormal basis vectors $$u$$ and $$v$$, and $$B^*$$,
the $$(n-2)$$-dimensional dual of $$B$$.

$$
B = u \wedge v  \\
B^* = B I^{-1}
$$

For the cut test, we use a region bounded by $$(n-3)$$D
hyperplanes of the dual space. Each of these hyperplanes
is the projection of a surface from the axis-aligned
$$n$$D unit hypercube. We can group these into sets of parallel
surfaces; parallel surfaces project into parallel hyperplanes,
and parallel hyperplanes share a common check axis.
Each set of parallel $$(n-3)$$D surfaces in the hypercube is
parallel to an $$(n-3)$$-blade composed of $$n-3$$ of the
elementary basis vectors, so there are $$n \choose n-3$$ such sets.

Let $$S$$ be the $$(n-3)$$-blade that is the combination of all elementary
basis vectors *except* for $$e_i$$, $$e_j$$, and $$e_k$$.
This is the same, up to sign, as the dual of $$e_{ijk}$$. Choose
the sign so that $$S^* = e_{ijk}$$. (I don't think the sign matters,
because we find a min and max bound for each check axis and we
don't really care which is the min and which is the max.)

$$
S = \pm e_1 \wedge \cdots
    \wedge \check e_i \wedge \cdots
    \wedge \check e_j \wedge \cdots
    \wedge \check e_k \wedge \cdots
    \wedge e_n \\
S^* = S I^{-1} = e_{ijk}
$$

We want to find the vector in $$B^*$$ orthogonal to $$S$$.
This is exactly the contraction inner product from geometric algebra.

Let $$a_S$$ be the check axis for $$S$$. We use some standard identities
and the fact that $$S \wedge B = B \wedge S$$ because $$B$$ is a bivector.

$$
\begin{align*}
a_S &= S \rfloor B^* \\
    &= S \rfloor (B I^{-1}) = S \rfloor (B \rfloor I^{-1}) \\
    &= (S \wedge B) \rfloor I^{-1} \\
    &= (B \wedge S) \rfloor I^{-1} \\
    &= B \rfloor (S \rfloor I^{-1}) = B \rfloor (S I^{-1}) \\
    &= B \rfloor S^* \\
    &= B \rfloor e_{ijk} \\
\end{align*}
$$

We can break this down to get a form that's easy to compute
without requiring a geometric algebra library.

$$
\begin{align*}
B \rfloor e_{ijk}
    &= (u \wedge v) \rfloor e_{ijk} \\
    &= u \rfloor (v \rfloor e_{ijk}) \\
    &= u \rfloor (v_i e_{jk} - v_j e_{ik} + v_k e_{ij}) \\
    &= v_i (u \rfloor e_{jk}) - v_j (u \rfloor e_{ik}) + v_k (u \rfloor e_{ij}) \\
    &= v_i (u_j e_k - u_k e_j) - v_j (u_i e_k - u_k e_i) + v_k (u_i e_j - u_j e_i) \\
    &= u_j v_i e_k - u_k v_i e_j - u_i v_j e_k + u_k v_j e_i + u_i v_k e_j - u_j v_k e_i \\
    &= (u_k v_j - u_j v_k) e_i + (u_i v_k - u_k v_i) e_j + (u_j v_i - u_i v_j) e_k \\
\end{align*}
$$

And this is our check axis for $$i$$, $$j$$, and $$k$$.
Iterate over all choices of $$i$$, $$j$$, and $$k$$ to get all the check axes.
