---
title: Interprocedural Distributive Environment (IDE) example
date: 2026-01-26
author: Kait
description: an example of a non-distributive transfer function.
---
# IDE non-distributivity example

This definition is from [1]:

> **Definition 3.7.** An environment transformer
> $$f : \textit{Env}(D, L) \to \textit{Env}(D, L)$$
> is *distributive* (denoted by $\to^d$) iff for every $\textit{env}_1, \textit{env}_2, \ldots \in \textit{Env}(D, L)$,
> $$t\left(\sqcap_i \textit{env}_i\right) = \sqcap_i (t(\textit{env}_i)).$$

This means that the transformer, when applied to a combined lattice value, is
*as effective as* the transformer applied to individual lattice values. This
has some consequences. For example, the transformer cannot depend on two
different points (variables) within the environment.

To see a counterexample where this leads to non-distributivity, consider a
constant propagation across `x := y + z` with the transfer function `x |-> env(y) + env(z)`.
This could be applied to two different states, $\{y \mapsto 0, z \mapsto 1\}$ and $\{y \mapsto 1, z \mapsto 0\}$.
Individually, both of these states would transform to contain $\{x \mapsto 1\}$,
but this cannot happen if the states are joined beforehand---a joining
of the states would lead to $\top$ for both $y$ and $z$, preventing the realisation that
$x \mapsto 1$.

So, this is not distributive.

[1] M. Sagiv, T. Reps, and S. Horwitz, “Precise interprocedural dataflow analysis with applications to constant propagation,” Theoretical Computer Science, vol. 167, no. 1–2, pp. 131–170, 1996, doi: [10.1016/0304-3975(96)00072-2](https://doi.org/10.1016/0304-3975(96)00072-2).

