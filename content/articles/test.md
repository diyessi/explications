---
title: "Autodiff"
date: 2023-08-27T14:53:04-07:00
math: true
draft: true
---
# Coordinate Notation

The usual positional notation for tensor coordinates is cumbersome when deriving properties of arbitrary rank tensors. It is simpler to name the axes and assume there is some unspecified mapping between the names and positions.

A positional tensor coordinate $c$ is a tuple such as $(3, 5),$ where $c[0] = 3$ and $c[1] = 5.$ We can just as easily use names instead of positions, as in $d = (x:3, y:5)$ and $d[x] = 3$ and $d[y] = 5$. The positional tuple can be thought of as shorthand for the named tuple $(0:3, 1:5).$ With the names, the order does not matter, so $$(x:3,y:5) \equiv (y:5,x:3).$$

