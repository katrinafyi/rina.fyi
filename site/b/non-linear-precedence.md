---
title: does anyone know about non-linear use-site precedence?
date: 2026-03-04
author: Kait
description: does anyone know about non-linear use-site precedence?
---

# does anyone know about non-linear use-site precedence?

This is a grammar problem.

Suppose we have a language where variable references are optionally annotated
with their type and we have a right-associative `->` operator (you can think of it
as  "implies", if you want).
```
type  ::= ident
var   ::= ident | ident ":" type
expr  ::= expr1 | expr1 "->" expr
expr1 ::= var | "(" expr ")"
```
This is all very normal. We have introduced `expr1` to represent expressions
with higher precedence (in other words, a tighter/indivisible binding). We have
changed the left hand side of `->` to use `expr1` and the right is unchanged
to implement the right-associativity. This lets us unambiguously parse
`x -> y -> z` as `x -> (y -> z)`.

Suppose, now, that we will add a function type also using the arrow syntax, like
`t -> u`. This will conflict with the expression arrow, so we'll need to
add some parentheses and precedence to the `type` non-terminal:
```
type  ::= type1 "->" type
type1 ::= ident | "(" type ")"

var   ::= ident | ident ":" type1

expr  ::= expr1 | expr1 "->" expr
expr1 ::= var | "(" expr ")"
```
We make the sensible restriction that the type for a variable must either be an
ident or it must be parenthesised.

## use-site precedence?

Suppose we want to add a let expression where we can bind variables,
optionally with types, like `let x : int = ...` or `let x = ...`.
```
expr ::= "let" var "=" expr "in" expr | ...
```
In such a
statement, there's no risk of conflicting with an expression, so it would be
possible and convenient to let the `->` type appear without parentheses. We
would like to write:
```ocaml
let x : int -> int = ...
```
If we want to re-use the `var` rule (and we do), then this causes a problem.
The `var` appearing as a let variable *can* have a top-level arrow,
whereas the `var` appearing inside another expression cannot.


So, by "use-site precedence", I mean that at the use-site of the `var` rule, we
want to override the use of the `type1` rule with `type`.

The problem here is that we have precedence hierarchies within precedence
hierarchies. We would need to expose the precedence of `type` to users
of `var`, e.g.,
```
var  ::= ident | ident ":" type
var1 ::= ident | ident ":" type1
```
Note, also, that these rules cannot reference each other. If one of them referenced
the other (like `var ::= var1 | ...`), then this would lead to ambiguity because
`type1` is a subrule of `type`. If we were to write the rules like this,
then there would be duplication when handling the parsed result.

This is starting to smell a lot like polymorphism and propagating all the
implicit type (precedence) parameters. It would be nice if we could write
Something like `var[type1 <- type]`.

## non-linear precedence?

To see the need for "non-linear", suppose we add parentheses to the `var` rule.
This would let people write their arrow typed variables as `(x : t -> v)`
as well as `x : (t -> v)`. The grammar becomes:
```
type  ::= type1 "->" type
type1 ::= ident | "(" type ")"

var   ::= ident | ident ":" type1 | "(" var ":" type ")"

expr  ::= expr1 | expr1 "->" expr
expr1 ::= var | "(" expr ")"
```
Immediately, this causes a problem. The sequence `"(" ident ":" type1 ")"`,
appearing in expression position, is ambiguous because the parentheses could be
part of either `var` or `expr`.

This is hard to fix. We would need to say that within the rule `"(" expr ")"`,
the contained `expr` cannot be a bare `var`. Or, we would have to say that
witihn the `var` case of `expr1`, it cannot be the parenthesised variant.

Both of these options are difficult, because they require *excluding* a rule
with strong precedence (either the bare `var` expression or the parenthesised
`var` variable). With the traditional implementation of precedence rules, the
stronger precedence rule is always going to be a possibility in any rule of
lower precedence.

I don't really know how to fix this.

## related work

[Jeff Walker writes](https://blog.adamant-lang.org/2019/operator-precedence/)
about partial orders for precedence, with manual declaration of the order
between particular rule cases. This is good, but I'd really like to define the
precedence rules at use-site. I think this makes it more readable, as
information is embedded within the rule. It also avoids a need for labelling
each alternative.

He also notes that there are few algorithms for parsing such rules. I would be
very interested to see if there was a reasonable lowering for some notion of
non-linear use-site precedence to ordinary BNF (similar in spirit to
monomorphisation). I would be especially interested if this could be done
without complicating the resulting AST _too_ much.

The idea of use-site precedence is very similar to what Haskell does with its
[`showsPrec`](https://hackage-content.haskell.org/package/base-4.22.0.0/docs/Text-Show.html#t:Show)
function of the `Show` typeclass
The `showsPrec` function takes an integer precedence and its implementation
is expected to produce a string which is appropriately parenthesised for
the given precedence. An example ([from here](https://stackoverflow.com/a/43639618))
looks like this:
```haskell
instance Show Expr where
  showsPrec p e0 =
    case e0 of
     Const n -> showParen (p > 10) $ showString "Const " . showsPrec 11 n
     x :+: y -> showParen (p > 6) $ showsPrec 6 x . showString " :+: " . showsPrec 7 y
     x :-: y -> showParen (p > 6) $ showsPrec 6 x . showString " :-: " . showsPrec 7 y
     x :*: y -> showParen (p > 7) $ showsPrec 7 x . showString " :*: " . showsPrec 8 y
     x :/: y -> showParen (p > 7) $ showsPrec 7 x . showString " :/: " . showsPrec 8 y
```
Intuitively, the outer `showParen` defines the "natural" precedence of the
returned string (wrapping it in parens if higher is needed), and each recursive
`showsPrec` call is annotated with the precedence of its use-site. Also note
the different precedence between left and right recursive calls, due to
associativity.
