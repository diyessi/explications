---
title: "Tensor Operations"
date: 2023-08-27T14:53:04-07:00
math: true
draft: true
---
# Tensors

The word *tensor* is used in a number of related but different ways. In mathematics, a tensor is a multi-linear relation between vector spaces. Given particular bases for the vector spaces, a tensor can be represented by a multi-dimensional array. In machine learning, *tensor* became a synonym for data that can be represented as a *multi-dimensonal array,* for multi-dimensional arrays in general, and for particular implementations of multi-dimensional arrays. Here we will treat a tensor as a function from *coordinate system* to an *element type.* 

Recall that a function from $A$ (its domain) to $B$ (its range) pairs every element of $A$ with an element of $B$. If two descriptions of functions result in the same pairings, the two descriptions describe the same function.

## Coordinate Systems

The ambiguity of the word *tensor* is nothing compared to the abiguity of words like *axis,* *dimension* and *coordinate.* Here we will introduce *typed axes* to simplify the definition of tensor operations and their properties.

Here, an *axis* is a *nominal type* for a set of values. By *nominal type* we mean that every axis definition is a new axis, even if the set of values is the same as another axis. We must be careful to distinguish an axis from an axis variable, since axis variables with the same name may refer to the same axis.

The set of values for an axis can be quite varied. We'll start by borrowing some convenient conventions from mathematical logic: A non-negative integer names the set of non-negative integers below it.
$$0 \equiv \\{\\},$$
$$1 \equiv \\{0\\} = 0 \cup \\{0\\},$$
$$2 \equiv \\{0, 1\\} = 1 \cup \\{1\\},$$
$$3 \equiv \\{0, 1, 2\\} = 2 \cup \\{2\\},$$
$$\vdots$$
$$n+1 \equiv \\{0, 1, \ldots, n\\} = n \cup \\{n\\}.$$
These are called *the von Neumann integers.*

We can define an axis as $\mathtt{axis\ }\mathtt{Name}[\mathtt{values}]$. For example, $\mathtt{axis\ }H[128]$ and $\mathtt{axis\ }W[128]$ defines two separate axes, $H$ and $W$, each with values from $0$ through $127.$ A *coordinate* pairs an axis with an axis value, for example, $H:4$ or $W:45.$

A *coordinate system* is specified by a set of axes. For example, $\\{H,W\\}$ specifies a coordinate system. Coordinate systems contain all sets of coordinates that contain exactly one of each axis in the coordinate system. For $\\\{H,W\\},$
$$\\{H:0,W:0\\},$$
$$\\{H:0:W:1\\},$$
$$\\{H:0:W:2\\},$$
$$\vdots$$
$$\\{H:127,W:127\\}.$$
Since a coordinate system is specified by a set of axes, the order doesn't matter: $\\{H,W\\}$ and $\\{W,H\\}$ specify the same coordinate system. Likewise, $\\{H:3,W:6\\}$ and $\\{W:6,H:3\\}$ are the same coordinate.

We can specify a tensor with its element type and coordinate system:
$$\mathtt{tensor\ float \ }T[\\{H,W\\}]$$
and an element as:
$$T[\\{W:4, H:6\\}].$$

This notation can get verbose, so there is also a positional variant:
$$\mathtt{tensor\ float \ }T[H,W] \equiv T[\\{H,W\\}]$$
and defines
$$T[4, 6] \equiv T[\\{W:4, H:6\\}].$$
All axes must be unique.

## Broadcasting

A tensor's data can be broadcast into a tensor with an expanded coordinate system by ignoring the axes that are only in the expanded coordinate system. If coordinate system $C, D$ are disjoint, we project coordinates from $C\cup D$ into $C:$
$$\begin{aligned}
\mathtt{tensor\ }U[C\cup D] &= \mathtt{Broadcast(D)}(T[C])\\\\
U[c\cup d] &=T[c].\\\\
\end{aligned}$$

If the coordinate system of the actual data is $\\{\\}$ (scalar) the coordinate system can be broadcast to $\\{H,W\\}$ so it can be used in settings where there are two axes, $H$ and $W$.

## Flattening and Inflating

A coordinate system may be *flattened* by treating its coordinates as the values of a new axis. For example, with the coordinate system $\\{N,H,W,C\\}$, we can flatten the subcoordinate system $\\{H,W,C\\}$ as $\texttt{axis\  }I[\\{H,W,C\\}],$ creating a new axis $I$. Alternatively, we can leave the axis unnamed, as $\texttt{flatten}(\\{H,W,C\\}),$ which is an unnamed new axis that is the same axis as any other use of $\texttt{flatten}(\\{H,W,C\\}).$

$$\begin{aligned}
U[I,J]&=\texttt{flatten}\\{I,J\\}T[I|J]\\\\
U[i,j]&=T[i|j].\\\\
\\end{aligned}$$

For example,
$$\begin{aligned}
U[N,\\{H,W,C\\}]&=\texttt{flatten}\\{N,\\{H,W,C\\}\\}(T[N,H,W,C])\\\\
U[n,\\{h,w,c\\}]&=T[n,h,w,c].\\\\
\end{aligned}$$

## Tensor Addition

NumPy positional tensor addition has three variations:
 1. $$\begin{aligned} V[I] &= T[I] + U[I]\\\\
      V[i] &= T[i] + U[i],\\\\
    \end{aligned}\text{   (elementwise)}$$
 2. $$\begin{aligned} V[I] &= T[1] + U[I]\\\\
      V[i] &= T[0] + U[i],\\\\
     \end{aligned}\text{  (broadcast single axis)}$$
 3. $$\begin{aligned}V[I,J] &= T[1,I] + U[J,1]\\\\ 
     V[i,j] &= T[1,0] + U[j,0],\\\\
     \end{aligned}\text{  (broadcast both axes)}$$
where $i\in I$ and $j\in J$.

With axis types, we can define the three types of addition for arbitrary axes. Let $I, J,$ and $K$ be three arbitrary disjoint coordinate sets, any of which may be empty. Define
$$\begin{aligned}
V[I\cup J \cup K] &= T[I \cup J]+U[J\cup K]\\\\
V[i \cup j \cup k] &= T[i \cup j]+U[j\cup k]\\\\
\end{aligned}
$$
where $i\cup j \cup k\in I\cup J\cup K$.

Positional case 1 applies when $I$ and $K$ are empty, case 2 when $I$ and $J$ are empty and case 3 when $J$ is empty.

To simplify the notation, we let $I|J$ be a disjoint union of coordinate spaces, and $i|j$ be the corresponding separation of their coordinates. Then we can rewrite addition more concisely as:
$$\begin{aligned}
V[I|J|K] &= T[I|J]+U[J|K]\\\\
V[i|j|k] &= T[i|j]+U[j|k].\\\\
\end{aligned}
$$



## Projections

Sometimes we need to project a coordinate onto a smaller coordinate set. If $hwc$ is a coordinate in $\\{H,W,C\\}$ then $\\{H,W\\}(hwc)$ will retain the locations on the $H$ and $W$ axis but not the location on the $C$ axis. For example,
$$\\{h:3,w:1\\} = \\{H,W\\}({h:3, w:1, c:2}).$$

# Basic Tensor Operations

## Tensor Addition

We can now define tensor addition. Given $\mathtt{Tensor\ }T[A]$ and $\mathtt{Tensor\ }U[B]$, the tensor sum will be $\mathtt{Tensor\ }V[A,B]$, where for all coordinates $ab \in A,B$,
$$V[ab] = T[A(ab)] + U[B(ab)].$$

Equivalently, given disjoint coordinate systems $A, B,$ and $C$, and $\mathtt{Tensor\ }T[A,B]$ and $\mathtt{Tensor }U[A,C]$, $$\mathtt{Tensor\ }V[A|B|C] = T[A|B] + U[A|C]$$ where
$$V[a|b|c] = T[a|b]+ U[a|c].$$

The sum has all the axes of the two tensors being summed. The value at each coordinate in the sum is the sum of the values of the two tensors at the coordinate projected into their coordinate system.

How does this compare with positional tensor addition? Transpositions may be required to make axes positions line up correctly, and axes of length 1 may need to be added to one or both tensors to indicate broadcasts on the appropriate axes.

### Derivative

For each element in $V$,
$$dV[a|b|c] = dT[a|b] + dU[a|c].$$

Then
$$dV = dT + dU.$$

## Reduction

If $A$ and $B$ are disjoint coordinate systems and $\mathtt{Tensor\ }T[A|B]$ is a tensor, the reduction of T on $A$ is $\mathtt{Tensor\ }U[B]$ where for each $b\in B$,
$$U[b]=\sum_{a\in A} T[a|b].$$

### Derivative

$$dU[b] = \sum_{a\in A} dT[a|b].$$
$$dU = \sum_{a\in A} dT.$$

## Tensor Multiplication

If $A, B,$ and $C$ are disjoint coordinate systems, the product of $T[A|B]$ and $U[B|C]$ is $V[A|C]$ where for all $a\in A$ and $b\in B$,
$$V[a|c] = \sum_{b\in B} A[a|b]B[b|c].$$

### Derivative
$$dV[a|c] = \sum_{b\in B} \left(A[a|b]dB[b|c] + dA[a|b]B[b|c]\right).$$
$$dV = AdB + BdA.$$

## Batched multiplication

If the shared coordinates $B$ are split into two groups, $B_0$ and $B_1$, we can define a batched multiplication:
$$V[B_0|A|C] = T[B_0| A| B_1] U[B_0| B1| C],$$
$$V[b_0| a|c] = \sum_{b_1\in B_1} A[b_0| a|b_1]B[b_0| b_1|c].$$

### Derivative

$$V[b_0| a|c] = \sum_{b_1\in B_1} \left(A[b_0| a|b_1]dB[b_0| b_1|c]+dA[b_0| a|b_1]B[b_0| b_1|c]\right).$$
$$dV[B_0|A|C] = A[B_0|A|B_1]dB[B_0|B_1|C] + dA[B_0|A|B_1]B[B_0|B_1|C].$$