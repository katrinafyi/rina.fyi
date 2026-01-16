---
title: Interprocedural Distributive Environment (IDE) examples
date: 2026-01-26
author: Kait
description: pointers to resources which helped me learn Nix flakes.
---
# IDE problem examples

> **Definition 3.7.** An environment transformer
> $$f : \textit{Env}(D, L) \to \textit{Env}(D, L)$$
> is *distributive* (denoted by $\to^d$) iff for every $\textit{env}_1, \textit{env}_2, \ldots \in \textit{Env}(D, L)$,
> $$t\left(\sqcap_i \textit{env}_i\right) = \sqcap_i (t(\textit{env}_i)).$$

Intuitively, this means that the transformer, when applied to a combined
lattice value is *as effective* as the transformer applied to individual
lattice values. This has some consequences, such as the transformer cannot
depend on two different points (variables) within the environment.

