---
title: "Automatic Differentiation for Deep Learning"
date: 2023-08-27T14:53:04-07:00
math: true
draft: true
---
# Introduction

When most people hear the term "Automatic differentiation," they think it is the same process learned in calculus only performed by a program. Others think it is a numeric approximation to the actual derivative. In reality, it is a clever transformation that extends a program that computes a function into one that also computes its derivatives with respect to specified parameters. Here we will provide a brief review of calculus followed by an explanation of automatic differentiation and some applications to deep learning computations.

# A Short Vector Calculus

The derivative is a linear approximation to a function at a particular value. In deep learning, the function is a cost function composed with an iterated inference computation that is the composition of many machine learning operations which are themselves the composition of many arithmetic operations. The linear approximation is an approximation of how the result of the computation would vary if the tunable parameters were varied slightly as a group, including which combined variation will make the greatest incremental improvement.

This is loosely based on [Michael Spivak's *Calculus on Manifolds*](https://www.amazon.com/Calculus-Manifolds-Approach-Classical-Theorems/dp/0805390219), [R. M. Dudley's *Real Analysis and Probability*](https://www.amazon.com/Analysis-Probability-Cambridge-Advanced-Mathematics/dp/0521007542), and lectures by R. M. Dudley and James Munkres.

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

### Matrix Multiplication

Now suppose that $f:\mathcal{V}(C_0)\rightarrow\mathcal{V}(C_1)$ is linear. We can completely characterize $f$ with a vector $F\in\mathcal{V}(C_1\times C_0)$ by letting
$$F[i,j]=f(1_j)[i].$$ Then
$$\begin{align*}
f(v)[i]&=f(\sum_{j\in C_0} v[j]1_j)[i]\\\\
&=v[j](\sum_{j\in C_0}f(1_j)[i])\\\\
&=v[j]\sum_{j\in C_0}F[i,j]\\\\
&=\sum_{j\in C_0}F[i,j]v[j],
\end{align*}$$
which is just matrix multiplication with more general coordinates.
We write $$f(v)=Fv.$$
So every linear vector function corresponds to a matrix multiplication and it is easy to verify that every matrix multiplication corresponds to a linear function.

### Composition

If we have $f:\mathcal{V}(C_0)\rightarrow \mathcal{V}(C_1)$ and $g:\mathcal{V}(C_1)\rightarrow\mathcal{V}(C_2)$
$$\begin{align*}
g(f(s_0x_0+s_1x_1))&=g(s_0f(x_0)+s_1f(x_1))\\\\
&=s_0g(f(x_0))+s_1g(f(x_1)),
\end{align*}$$
so the composition of linear functions is linear. Furthermore if $f(v)=Fv$ and $g(u)=Gu$ then $f(g(v))=GFv$ where
$$(GF)[i,k]=\sum_{j\in C_1}G[i,j]F[j,k],$$
the definition of matrix multiplication. Matrix multiplication is associative and distributive, but in general is not commutative.

### Distance

For $v_1, v_2\in \mathcal{V}(C)$, the inner product is defined as
$$v_1\cdot v_2 = \sum_{i\in C} v_1[i]v_2[i].$$
The length of $v\in \mathcal{V}(C)$ is
$$|v|=\sqrt{v\cdot v}.$$ The distance between $v_1$ and $v_2$ is $d(v_1, v_2) = |v_1-v_2|.$

### Scalars as Tensors

We equate $\mathcal{V}(\\{\\})$ with the scalars.

### Cauchy Sequences

A sequence $s$ is a function whose domain is $\mathbb{N}.$ We write $s_i$ for the value of the function at $i.$ A Cauchy sequence is a sequence where for every real $\epsilon > 0$ there is an $N\in\mathbb{N}$ such that for all $m,n> N$, $d(s_m,s_n) < \epsilon.$ In other words, if you pick any positive distance, at some point all further elements of the sequence stay within that distance of each other.

If you make a new sequence from two Cauchy sequences by alternating elements and the new sequence is still a Cauchy sequence, then you can see that in some sense they are both "heading in the same direction." And if one of those sequences is a constant sequence, i.e. all elements are the same, then we say that both sequences converge to that constant value.

Not all Cauchy sequences of rational numbers converge. For example, a Cauchy sequence of approximation of $\sqrt{2}$ cannot converge because $\sqrt{2}$ is not rational. But for every Cauchy sequence of rationals there are many other sequences that "head in the same direction." An example of three sequences that "head in the same direction" are
$$\begin{align*}
&.9, .99, .999, .9999, \ldots\\\\
&1.0, 1.0, 1.0, 1.0, \ldots\\\\ 
&1.1, 1.01, 1.001, 1.0001, \ldots\\\
\end{align*}$$
Every real number can be defined of as representing all the Cauchy sequences that "head in the same direction," whether or not they converge to a rational. In the above example, all three sequences converge to $1$, by definition. It turns out that every Cauchy sequence of reals converges to a real.

Since we have a distance function for vectors, vectors can also have Cauchy sequences, and all Cauch sequences of real vectors converge to a real vector.

### Limits

Our definition of the derivative for vector functions is going to involve
$$g=\lim_{|h|\rightarrow 0}f(h)$$
for vectors $h.$ This is defined to mean that if we pick any $\epsilon > 0$ we can find a $\delta$ such that whenever $|h|<\delta$ then $|g-f(h)|<\epsilon.$ Given these conditions, we can form Cauchy sequences, guaranteeing that there is actually a real vector $g$ that satisfies these conditions.

### Derivative

Given a function $f:\mathcal{V}(C_0)\rightarrow \mathcal{V}(C_1)$ and $x\in \mathcal{V}(C_0)$, if there is a matrix $Df(x \in\mathcal{V}(C_1\times C_0)$ such that for $h\in\mathcal{V}(C_0),$
$$\lim_{|h|\rightarrow 0}\frac{f(x+h)-f(x)-(Df(x))h}{|h|}=0,$$
It can be shown that if there is such a matrix, it is unique. $Df(x)$ is called the derivative of $f$ at $x$. It is a linear approximation to the change in $f(x)$ near $f(x).$

Recall our definition of limit and how we determined a matrix for a linear function. All ways that $|h|$ can approach 0 must result in the matrix $Df'(x).$ For $j\in C_0$ we can approach 0 with $h1_j$, for $h\in\mathbb{R}.$ and we can look individually at coordinate $i\in C_1$, to get
$$\lim_{|h|\rightarrow 0}\frac{f(x+h1_j)[i]-f(x)[i]-((Df(x))h1_j)[i]}{|h|}=0.$$
Now $h$ is real, as is $f(\cdot)[i]$, so this is an ordinary derivative of a real function, denoted by
$$\frac{\partial f_i}{\partial x_j}.$$
Then
$$Df(x)[i,j] = \frac{\partial f_i}{\partial x_j}.$$

For example, let
$$[z_0, z_1, z_2]=[x^2, xy, y^2],$$
so $C_0=\\{x,y\\}$ and $C_1=\\{z_0, z_1, z_2\\}.$
Then
$$
\begin{align*}
\frac{\partial z_0}{\partial x} &=2x\\\\
\frac{\partial z_0}{\partial y} &=0\\\\
\frac{\partial z_1}{\partial x} &=y\\\\
\frac{\partial z_1}{\partial y} &=x\\\\
\frac{\partial z_2}{\partial x} &= 0\\\\
\frac{\partial z_2}{\partial y} &= 2y.
\end{align*}
$$
and the derivative is
$$
\left[\begin{array}{cc}
2x&0\\\\
y&x\\\\
0&2y
\end{array}\right].
$$

### The Chain Rule

Earlier we showed that the composition of linear functions is a linear function. The derivative of the composition of two functions is the composition of the derivatives, i.e. the matrix product.

# Automatic Differentiation

This is loosely based on [Andrea Griewank and Andrea Walther's *Evaluating Derivatives: Principles and Techniques of Algortihmic Differentiation 2nd Edition*](https://www.amazon.com/Evaluating-Derivatives-Principles-Algorithmic-Differentiation/dp/0898716594).

Let
$$z=ax^2+bxy+cy^2.$$
In a typical calculus problem we would want $\frac{\partial z}{\partial x}$ and $\frac{\partial z}{\partial y},$ but in deep learning we would be trying to optimize by adjusting $a, b$ and $c$, so we would want $\frac{\partial z}{\partial a},$ $\frac{\partial z}{\partial b},$ and $\frac{\partial z}{\partial c}.$

## The Computation

To actually compute $z$ given vaues for $a,b,c,x,$ and $y,$ we will need to break the expression down into a series of primitive operations. The network has three primitive operations, scalar addition, multiplication, and squaring. We will assume that the parameters $a,b,$ and $c$ and the arguments $x$ and $y$ must be loaded before being used in computation, and the the result must be saved.

```python
t0 = parameter.a
t1 = arg.x
t2 = t1^2
t3 = t0 * t2
t4 = parameter.b
t5 = t4 * t1
t6 = arg.y
t7 = t5 * t6
t8 = t3 + t7
t9 = parameter.c
t10 = t6^2
t11 = t9 * t10
t12 = t8 + t11
result.z = t12
```

## Differentials of Primitives

For addition,
$$\begin{align*}
z&=x+y\\\\
dz&=dx+dy\\\\
dz&=\left[\begin{array}{cc}
1&1
\end{array}\right]
\left[\begin{array}{c}
dx\\\\
dy
\end{array}\right]
\end{align*}$$

For multiplication,
$$\begin{align*}
z&=xy\\\\
dz&=x\\,dy + y\\,dx\\\\
dz&=\left[\begin{array}{cc}
y&x
\end{array}\right]
\left[\begin{array}{c}
dx\\\\
dy
\end{array}\right]
\end{align*}$$

And for squaring,
$$\begin{align*}
z&=x^2\\\\
dz&=2x\\\\
dz&=\left[\begin{array}{c}
2x\\,dx
\end{array}\right]
\left[\begin{array}{c}
dx
\end{array}\right]
\end{align*}$$

## Computing the Derivative

We will start with the forward computation of the derivative. We want to compute $\frac{\partial z}{\partial a},$ $\frac{\partial z}{\partial b},$ and $\frac{\partial z}{\partial c}.$ We use vectors of the form
$$\left[\begin{array}{ccc}\frac{\partial}{\partial a}&\frac{\partial}{\partial b}&\frac{\partial}{\partial c}\end{array}\right].$$
After we compute `ti` we compute `Di` which is the derivative matrix, followed by `dti` which is the value of the differential vector.  while for `arg` operations it is 0. For addition, subtraction and squaring it is result of the above differential computations.

```python
t0 = parameter.a
dt0 = [1, 0, 0]
```
Since $\frac{\partial a}{\partial a} = 1$ and $\frac{\partial a}{\partial b}, \frac{\partial a}{\partial c} = 0$ 

```python
t1 = arg.x
dt1 = [0, 0, 0]
```
Since $\frac{\partial x}{\partial a} = 0.$

```python
t2 = t1^2
D2 = [2 * t1]
dt2 = D2[dt1]
```
Here we encounter the first derivative matrix, this time for the unary function $x^2.$

```python
t3 = t0 * t2
D3 = [t2, t0]
dt3 = D3[dt0 
         dt2]
```
Multiplication is binary, so we have contributions from both arguments.

```python
t4 = parameter.b
dt4 = [0, 1, 0]

t5 = t4 * t1
D5 = [t1, t4]
dt5 = D5[dt4
         dt1]

t6 = arg.y
dt6 = [0, 0, 0]

t7 = t5 * t6
D7 = [t6, t5]
dt7 = D7[dt5
         dt6]

t8 = t3 + t7
D8 = [1, 1]
dt8 = D8[dt3
         dt7]

t9 = parameter.c
dt9 = [0, 0, 1]

t10 = t6^2
D10 = [2 * t6]
dt10 = D10[dt6]

t11 = t9 * t10
D11 = [t10, t9]
dt11 = D11[dt9,
           dt10]

t12 = t8 + t11
D12 = [1, 1]
dt12 = D12[dt8
           dt11]

result.z = t12
result.dz = dt12
```

Now we'll evaluate symbolically:
```python
t0 = parameter.a = a
dt0 = [1, 0, 0]

t1 = arg.x = x
dt1 = [0, 0, 0]

t2 = t1^2 = x^2
D2 = [2 * t1] = 2x
dt2 = D2[dt1] = [0, 0, 0]

t3 = t0 * t2 = ax^2
D3 = [t2, t0] = [x^2, a]
dt3 = D3[dt0  = [x^2, a][1, 0, 0
         dt2]            0, 0, 0]
              = [x^2, 0, 0]

t4 = parameter.b = b
dt4 = [0, 1, 0]

t5 = t4 * t1 = bx
D5 = [t1, t4] = [x, b]
dt5 = D5[dt4  = [x, b][0, 1, 0
         dt1]          0, 0, 0]
              = [0, x, 0]

t6 = arg.y = y
dt6 = [0, 0, 0]

t7 = t5 * t6 = bxy
D7 = [t6, t5] = [y, bx]
dt7 = D7[dt5  = [y, bx][0, x, 0
         dt6]           0, 0, 0]
              = [0, xy, 0]

t8 = t3 + t7 = ax^2 + bxy
D8 = [1, 1]
dt8 = D8[dt3  = [1, 1][x^2, 0,  0
         dt7]          0,   xy, 0]
              = [x^2, xy, 0]

t9 = parameter.c = c
dt9 = [0, 0, 1]

t10 = t6^2 = y^2
D10 = [2 * t6] = [2y]
dt10 = D10[dt6] = [2y][0, 0, 0]
                = [0, 0, 0]

t11 = t9 * t10 = cy^2
D11 = [t10, t9] = [y^2, c]
dt11 = D11[dt9,  = [y^2, c][0, 0, 1
           dt10]            0, 0, 0]
                 = [0, 0, y^2]

t12 = t8 + t11 = ax^2 + bxy + cy^2
D12 = [1, 1]
dt12 = D12[dt8   = [1, 1][x^2, xy, 0
           dt11]            0,  0, y^2]
                 = [x^2, xy, y^2]

result.z = t12
result.dz = dt12 = [x^2, xy, y^2]
```


# Deep Learning Operations