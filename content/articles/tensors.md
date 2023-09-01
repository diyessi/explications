---
title: "Tensor Operations"
date: 2023-08-27T14:53:04-07:00
math: true
draft: true
---
# Coordinates

Tensors provide access to items of data identified by a set of coordinates. Each coordinate specifies a value on an axis. Axes are usually assigned positions, such as $[X, Y, Z]$. Then the values in the tuple $[2, 5, 1]$ can be matched to the axes in the same positions, so $2$ is the coordinate on the $X$ axis, $5$ is the coordinate on the $Y$ axis, and $1$ is the coordinate on the $Z$ axis.

For tensor operation semantics, it is the axes, not the positions, that are important. This is not to say that positional notation is not important for implementing tensors and their operations, and it is considerably more concise for using tensors. But for defining the operations and their properties it is much simpler to skip positions and work with axes directly.

## Axis

We define an *axis* by naming a set of values used for coordinates. For example, $\mathtt{Axis\ } X[\mathbb{R}]$ is named $X$ and its coordinates are real numbers. A coordinate on $X$ is written $X:3.$ Axes are nominal, which means that each axis definition is a new axis even if the set of values is the same as another axis. However, variables with different names may refer to the same axis.

## Coordinate

Axis coordinates for tensors are usually bounded non-negative integers, such as $\\{0,1,2\\}.$ It is convenient to borrow the following definitions from mathematical logic:
$$0 \equiv \\{\\},$$
$$1 \equiv \\{0\\},$$
$$2 \equiv \\{0, 1\\},$$
$$\vdots$$
Then $\mathtt{Axis\ }H[128]$ would have coordinates $\\{0, 1, 2, \ldots, 127\\}.$

## Coordinate System

A *coordinate system* is a set of axes and a coordinate in a coordinate system is a set of coordinates with exactly one coordinate for each axis in the coordinate system. The coordinate
$$\\{H:3,W:4,C:2\\}$$
is in the coordinate system
$$\\{H[128], W[128], C[3]\\}.$$

## Nested Coordinate Systems

A coordinate system is a set of values, so a coordinate system can be used as the set of values for a new axis. For example, $\mathtt{axis }I[\\{H,W,C\\}]$ has coordinates such as $I:\\{H:2, W:4, C:0\\}.$

## Tensors

To write that the tensor $T$ has the coordinate system $\\{H,W,C\\}$ we write $\mathtt{Tensor\ }T[\\{H,W,C\\}]$. Likewise, $T[\\{H:3,W:1,C:0\\}]$ is the element of $T$ at coordinate $\\{H:3,W:1,C:0\\}.$

We can specify axes positionally by listing them: $$\mathtt{Tensor\ }T[C,H,W].$$ To indicate skipped positions (for broadcast) we use a $1$ instead of an axis: $$\mathtt{Tensor\ }T[1,C,H,W].$$

## Disjoint Coordinate Systems

If we write that tensor $T$ has coordinate system $\mathtt{Tensor\ }T[A,B,C]$ we mean that $A, B,$ and $C$ are disjoint coordinate systems and $T$'s coordinate system is $A\cup B\cup C$.

## Projections

Sometimes we need to project a coordinate onto a smaller coordinate set. If $hwc$ is a coordinate in $\\{H,W,C\\}$ then $\\{H,W\\}(hwc)$ will retain the locations on the $H$ and $W$ axis but not the location on the $C$ axis. For example,
$$\\{h:3,w:1\\} = \\{H,W\\}({h:3, w:1, c:2}).$$

# Basic Tensor Operations

## Tensor Addition

We can now define tensor addition. Given $\mathtt{Tensor\ }T[A]$ and $\mathtt{Tensor\ }U[B]$, the tensor sum will be $\mathtt{Tensor\ }V[A,B]$, where for all coordinates $ab \in A,B$,
$$V[ab] = T[A(ab)] + U[B(ab)].$$

Equivalently, given disjoint coordinate systems $A, B,$ and $C$, and $\mathtt{Tensor\ }T[A,B]$ and $\mathtt{Tensor }U[A,C]$, $$\mathtt{Tensor\ }V[A,B,C] = T[A,B] + U[A,C]$$ where
$$V[a,b,c] = T[a,b]+ U[a,c].$$

The sum has all the axes of the two tensors being summed. The value at each coordinate in the sum is the sum of the values of the two tensors at the coordinate projected into their coordinate system.

How does this compare with positional tensor addition? Transpositions may be required to make axes positions line up correctly, and axes of length 1 may need to be added to one or both tensors to indicate broadcasts on the appropriate axes.

## Reduction

If $A$ and $B$ are disjoint coordinate systems and $\mathtt{Tensor\ }T[A,B]$ is a tensor, the reduction of T on $A$ is $\mathtt{Tensor\ }U[B]$ where for each $b\in B$,
$$U[b]=\sum_{a\in A} T[a,b].$$

## Tensor Multiplication

If $A, B,$ and $C$ are disjoint coordinate systems, the product of $T[A,B]$ and $U[B,C]$ is $V[A,C]$ where for all $a\in A$ and $b\in B$,
$$V[a,c] = \sum_{b\in B} A[a,b]B[b,c].$$

If the shared coordinates $B$ are split into two groups, $B_0$ and $B_1$, we can define a batched multiplication:
$$V[B_0,A,C] = T[B_0, A, B_1] U[B_0, B1, C],$$
$$V[b_0, a,c] = \sum_{b_1\in B_1} A[b_0, a,b_1]B[b_0, b_1,c].$$

# Derivatives

