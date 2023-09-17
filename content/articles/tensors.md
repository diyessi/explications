---
title: "Automatic Differentiation for Deep Learning"
date: 2023-08-27T14:53:04-07:00
math: true
draft: true
---
# Introduction

When most people hear the term "Automatic differentiation," they think it is the same process learned in calculus only performed by a program. Others think it is a numeric approximation to the actual derivative. In reality, it is a clever transformation that extends a program that computes a function into one that also computes its derivatives with respect to specified parameters. Here we will provide a brief review of calculus followed by an explanation of automatic differentiation and some applications to deep learning computations.

# A Short Calculus

The derivative is a linear approximation to a function at a particular value. In deep learning, the function is a cost function composed with an iterated inference computation that is the composition of many machine learning operations which are themselves the composition of many arithmetic operations. The linear approximation is an approximation of how the result of the computation would vary if the tunable parameters were varied slightly as a group, including which combined variation will make the greatest incremental improvement.

## Vector Spaces

We need vector calculus whenever there is more than one parameter to tune. We don't need tensor calculus. Even though just about everything is called a tensor in machine learning, these are just multi-dimensional arrays and not the mathematical tensors of physics and tensor calculus. All we need are vectors, but it will be very convenient to enhance them.

### Coordinates

Here the natural numbers are $\mathbb{N}=\\{0, 1, 2, \ldots\\}$. When John von Neumann was working on the foundations of mathematics, he defined the integers as:
$$\begin{align*}
0&=\\{\\}\\\\
1&=\\{0\\}\\\\
2 &=\\{0,1\\}\\\\
&\vdots\\\\
n+1 &= \\{0, 1, \ldots, n-1\\} = n\cup \\{n\\}.\\\\
\end{align*}$$
In the foundations of mathematics, everything is a made from sets. Here we'll just assume the elements of $\mathbb{N}$ exist in some unspecified form and overload them as names for the subsets of $\mathbb{N}$ less than them.

To see how these von Neumann integers are convenient, consider the C++ vector declared as
```cpp
float x[5];
```
The "$5$" names the set of valid coordinates for `x`, $$\\{0,1,2,3,4\\}.$$

If you ignore the syntax and just look at the semantics, `x` represents the function $$x:5\rightarrow\mathbb{R}.$$ In fact, in C++, Python, and other languages you can define special functions to handle the operator `[ ]` in special ways.

We are going to need sets of coordinates that are more general than the von Neumann integers. To cover all eventualities, we'll let any set $C$ be available as a set of coordinates. We will call the set of functions
$$\mathcal{V}(C)=\\{v|v:C\rightarrow \mathbb{R}\\}$$
the vector space with coordinates $C$. Each function $v$ is a vector in this space.

To be a vector space, as opposed to just a set of vector, there must be a commutative and associative vector addition operation and a distributive scalar multiplication operation. It is easy to see that if $f,g\in \mathcal{C}$ then the usual function addition $f+g\in\mathcal{C}$ and is commutative and associative. Likewise, for $s\in \mathbb{R}$, the usual multiplication $sf\in\mathcal{C}$ and is distributive, so $\mathcal{V}(C)$ is a vector space.

So now $\mathcal{V}(2\times 3\times 5)$ is the vector space of tensors with shape $(2,3,5).$ $\mathcal{V}(\\{\mathtt{x}, \mathtt{y}\\})$ is a vector space of vectors with coordinates $\mathtt{x}$ and $\mathtt{y}$.

Any $n$-ary real function/operation $f$ extends to $n$-ary element-wise functions on combinations of scalars and tensors from $\mathcal{V}(C)$ to a result in $\mathcal{V}(C)$ by letting
$$v[i]=f(u_0[i], s_1, u_1[i], \ldots)$$
for $i\in C.$ This extension is compatible with the function addition and scalar multiplication we have already assumed.

For each coordinate $i\in C$ let $1_i$ be the tensor where
$$
1_i[j]=\begin{cases}
1&\text{if }i=j\\\\
0&\text{otherwise}
\end{cases}
$$
These form a basis for $\mathcal{V}(C).$

### Linear Functions

For vector spaces $X, Y$ a function $f:X\rightarrow Y$ is linear if for all $x_0, x_1\in X$ and $s_0, s_1\in\mathbb{R},$
$$f(s_0x_0+s_1x_1) = s_0f(x_0) + s_1f(x_1).$$

The simplest linear function is multiplication by a scalar. If $f(x)=ax$ then
$$f(s_0x_0+s_1x_1)=a(s_0x_0+s_1x_1)=s_0f(x_0)+s_1f(x_1).$$

If $f,g$ are linear so is their composition:
$$\begin{align*}
f(g(s_0x_0+s_1x_1))&=f(s_0g(x_0)+s_1g(x_1))\\\\
&=s_0f(g(x_0))+s_1f(g(x_1)).
\end{align*}$$

Now suppose that $f:\mathcal{V}(C_0)\rightarrow\mathcal{V}(C_1)$ is linear. We can completely characterize $f$ with a vector from $\mathcal{V}(C_1\times C_0).$

# Tensor Operations

For $v_1, v_2\in \mathcal{V}(C)$, the inner product is defined as
$$v_1\cdot v_2 = \sum_{i\in C} v_1[i]v_2[i].$$
The length of $v\in \mathcal{V}(C)$ is
$$|v|=\sqrt{v\cdot v}.$$







Tensor operations are function schemas whose inputs and outputs are tensors.

### Elementwise Operations

Elementwise operations extend a scalar function to tensors to create a new tensor where the value at each coordinate is the function applied to each tensor argument at the sameposition.

If tensors $T_1, T_2, \ldots, T_n$ all haves axes $A$ and element types $E_1, E_2, \ldots, E_n$ and $$f:E_1\times E_2\times \ldots \times E_n\rightarrow E$$ then we define $$f:T_1 \times T_2, \ldots, T_n \rightarrow \mathtt{tensor\ }E T[A]$$ where for $a\in A$,
$$T[a] = f(T_1[a], T_2[a], \ldots, T_n[a]).$$

### Axis Operations

Whereas the elementwise operations work with tensor values, axis operations work with the axes. Some simple examples:
 - Change the name of one or more axes,
 - For positional axes, permute the order (transpose)
 - Positional axes to axis set
 - Axis set to positional axes

#### Broadcast

#### Flatten

#### Inflate

#### Slice

#### Stack

#### Concatenate

## Coordinate Systems

The ambiguity of the word *tensor* is nothing compared to the abiguity of words like *axis,* *dimension* and *coordinate.* Here we will introduce *typed axes* to simplify the definition of tensor operations and their properties.

Here, an *axis* is a *nominal type* for a set of values. By *nominal type* we mean that every axis definition is a new axis, even if the set of values is the same as another axis. We must be careful to distinguish an axis from an axis variable, since axis variables with the same name may refer to the same axis.


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
U[\\{I,J\\}]&=\texttt{flatten}\\{I,J\\}T[I|J]\\\\
U[i,j]&=T[i|j].\\\\
\\end{aligned}$$

For example,
$$\begin{aligned}
U[\\{N,\\{H,W,C\\}\\}]&=\texttt{flatten}\\{N,\\{H,W,C\\}\\}(T[N,H,W,C])\\\\
U[\\{n,\\{h,w,c\\}\\}]&=T[\\{n,h,w,c\\}].\\\\
\end{aligned}$$

The opposite of flattening is inflating; expanding a flattened coordinate system back into the containing coordinate system.
$$\begin{aligned}
U[I|J]&=\texttt{inflate}\\{I|J}(T[I,J])\\\\
I[i|j]&=T[\{I,J}].\\\\
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