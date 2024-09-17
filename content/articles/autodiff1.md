---
title: "Automatic Differentiation"
date: 2023-10-03T04:00:00-07:00
math: true
---
# Introduction

When most people hear the term "Automatic differentiation," they think it is the same process learned in calculus only performed symbolically by a program. Others think it is a numeric approximation to the actual derivative. In reality, it is a clever transformation that extends a program that computes a function into one that also computes its derivatives with respect to specific parameters. Here we will provide a description of automatic differentiation by both forward and backward propagation.

# Scalar derivatives

We'll start with finding the derivative of a simple computation in three ways:
 - Symbolically, the way you learn in calculus,
 - Using forward propagation, the way automatic differentiation was originally performed.
 - Using back propagation, the way deep learning frameworks usually do it. Backwards propagation tends to be more efficient for computations where the number of computed values is small computed to the number of parameters.
 
The equation we'll use is:
$$f(x)=(x+a)(x+b).$$
This is a function of \\(x\\) with parameters \\(a\\) and \\(b.\\) We want to find \\(\frac{df}{da}\\) and \\(\frac{df}{db}\\) for a fixed \\(x\\).

## Symbolic differentiation

Symbolically,
$$\begin{align*}
df&=d((x+a)(x+a))\\\\
&=(x+a)\\,d(x+b) + (x+b)\\,d(x+a)\\\\
&=(x+a)\\,db + (x+b)\\,da.
\end{align*}$$

## Forward propagation

We begin by rewriting the computation \\(y=f(x)\\) as a series of steps, each performing a primitive operation:
```python
t0 = load('x')  # x
t1 = load('a')  # a
t2 = t0 + t1    # x + a
t3 = load('b')  # b
t4 = t0 + t3    # x + b
t5 = t2 * t4    # (x + a) * (x + b)
y = t5
```
Each \\(t_i\\) could hold the numeric value at this point in the computation, or it could just be a handle to a representation of the computation, depending on the implementation. After each step, we also compute the derivative of the intermediate result with respect to \\(a\\) and \\(b\\) as relevant. We use `dt2_da` for \\(\frac{dt_2}{da}.\\) To save space, for values such as \\(\frac{dt_0}{da}\\) that are \\(0\\) we omit `dt0_da` assuming it is \\(0\\) in any use.

```python
t0 = load('x')  # x
t1 = load('a')  # a
dt1_da = 1
t2 = t0 + t1    # x + a
dt2_da = 1
t3 = load('b')  # b
dt3_db = 1
t4 = t0 + t3    # x + b
dt4_db = 1
t5 = t2 * t4    # (x + a) * (x + b)
dt5_da = t2 * dt4_da + t4 * dt2_da
       = t2  # x + b
dt5_db = t2 * dt4_db + t4 * dt2_db
       = t4  # x + a
y = t5
dy_da = t4
dy_db = t2
```
So 
\\(\frac{dy}{da} = x + b\\) and \\(\frac{dy}{db} = x + a.\\)

## Backward propagation

As with forward propagation we begin by rewriting the computation \\(y=f(x)\\) as a series of primitive steps, setting \\(t_i\\) (written `ti`) to the result of each step:
```python
t0 = load('x')  # x
t1 = load('a')  # a
t2 = t0 + t1    # x + a
t3 = load('b')  # b
t4 = t0 + t3    # x + b
t5 = t2 * t4    # (x + a) * (x + b)
y = t5
```
Now, for every \\(t_i\\) we add a \\(\overline{dt_i}\\), which we'll write as `dti = 0,` that will accumulate contributions.
```python
dt0 = 0
dt1 = 0
dt2 = 0
dt3 = 0
dt4 = 0
dt5 = 0
```
Next we let \\(\overline{dy}\\) be \\(1\\):
```python
dy = 1
```
Now we work backwards through the original computation. The derivative of each line will be of the form \\(dt_i=\sum_{j < i} a_jdt_j\\). For each term in the derivative, we add a computation
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\overline{dt_j}\pluseq a_j\overline{dt_j},
$$
starting with `y = t5:`
```python
dt5 += dy  # 1
```
Now we move to the next unprocessed operation, \\(t_5=t_2t_4.\\) Taking the derivative gives \\(dt_5 = t_2dt_4+t_4dt_2,\\) so we get two steps:
```python
dt4 += dt5 * t2 # x + a
dt2 += dt5 * t4 # x + b
```
Continuing,
```python
dt0 += dt4  # x + a
dt3 += dt4  # x + a (dy/db)
dt0 += dt2  # 2 * x + a + b (dy/dx)
dt1 += dt2  # x + b (dy/da)
```
So \\(\frac{dy}{da}=x+b\\) and \\(\frac{dy}{db}=x+a.\\)

If you think it is a bit mysterious how backwards propagation ends up with the correct values, your intuition is correct. If is often explained as just being the chain rule, along with some vigorous hand waving. Of course the chain rule is involved, but it is more interesting than just the chain rule. A future page will explain why it works.

# Tensor derivatives

For a scalar \\(t\\), we treat \\(dt\\) as a scalar. For a tensor \\(t\\) we treat \\(dt\\) as a tensor with the same shape and wor non-negative integers, \\(I, J, \ldots\\), \\(t[I,J, \ldots]\\) designates a tensor of the given shape, while for \\(i\in I, j\in j, \ldots, t[i,j,\ldots]\\) designates an element of the tensor. 

\\(t[i,J,\ldots]\\) is a slice of the tensor.

## Vector addition

For a size \\(I\\),
$$y[I] = u[I] + v[I]$$
is defined by
$$y[i] = u[i] + v[i],$$
we see that
$$dy[i] = du[i] + dv[i],$$
so for forward propagation,
$$dy[I] = du[I] + dv[I].$$
and for backward propagation,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{du[I]} &\pluseq \overline{dy[I]}\\\\
\overline{dv[I]} &\pluseq \overline{dy[I]}.
\end{aligned}$$

## Elementwise operations

Vector addition is a specific case of an elementwise operation. An \\(n\\)-ary function \\(f\\) is applied to \\(n\\) tensors, all with the same shape.
$$\begin{aligned}
y[I] &= f(x_0[I], x_1[I], \ldots)\\\\
y[i] &= f(x_0[i], x_1[i], \ldots)
\end{aligned}$$
Taking the derivative gives the forward propagation:
$$\begin{aligned}
dy[i] &= \sum_j \frac{\partial f(x_0[i], x_1[i], \ldots)}{\partial x_j} dx_i[i]\\\\
&=\sum_j \frac{\partial f(x_0, x_1, \ldots)}{\partial x_j} dx_i,
\end{aligned}$$
so, for backwards propagation,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\overline{dx_i}\pluseq \frac{\partial f(x_0, x_1, \ldots)}{\partial x_i}\\,\overline{dy}
$$

## Relu

\\(\mathop{\mathtt{relu}}\\) is a unary elementwise operation with a condition in it:
$$\begin{aligned}
Y[I]&=\mathop{\mathtt{relu}}(X[I])\\\\
y[i]&=\begin{cases}
x[i]&\text{if }x[i]\ge 0\\\\
0&\text{otherwise.}
\end{cases}
\end{aligned}$$
Because of the conditional in the definition, the derivative will also have a conditional. For forward propagation,
$$dy[i]=\begin{cases}
1&\text{if }x[i]\ge 0\\\\
0&\text{otherwise.}
\end{cases}
$$
Mathematically, the derivative is not defined at 0, so the forward propagation would be \\(\mathtt{NaN}\\) which would cause problems during training. Picking an arbitrary value will cause \\(x[i]\\) to move away from \\(0\\).

For backwards propagation,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\overline{dx[i]}\pluseq\begin{cases}
\overline{dy[i]}&\text{if }x[i]\ge 0\\\\
0&\text{otherwise.}
\end{cases}
$$

## Max

\\(\mathop{\mathtt{max}}\\) is a binary elementwise operation with a condition in it:
$$\begin{aligned}
Y[I]&=\mathop{\mathtt{max}}(X_0[I], X_1[I])\\\\
y[i]&=\begin{cases}
x_0[i]&\text{if }x_0[i]\ge x_1[i]\\\\
x_1[i]&\text{otherwise.}
\end{cases}
\end{aligned}$$
Because of the conditional in the definition, the derivative will also have a conditional. For forward propagation,
$$dy[i]=\begin{cases}
dx_0[i]&\text{if }x_0[i]\ge x_1[i]\\\\
dx_1[i]&\text{otherwise.}
\end{cases}
$$
Some people worry about the case where \\(x_0[i]=x_1[i].\\) Mathematically, when \\(x_0[i]=x_1[i],\\) the derivative is only defined when \\(dx_0[i]=dx_1[i]\\) so the "correct" answer would be \\(\mathtt{NaN}\\). By arbitrarily picking one, training will adjust \\(x_0[i]\\), breaking the tie.

and for backwards propagation,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx_0[i]}&\pluseq\begin{cases}
\overline{dy[i]}&\text{if }x_0[i]\ge x_1[i]\\\\
0&\text{otherwise.}
\end{cases}\\\\
\overline{dx_1[i]}&\pluseq\begin{cases}
\overline{dy[i]}&\text{if }x_0[i] < x_1[i]\\\\
0&\text{otherwise.}
\end{cases}
\end{aligned}
$$


## General coordinates

Everything above in vector addition and elementwise operations can be extended to matrices by changing each '\\(I\\)' to '\\(I,J\\)' and changing each '\\(i\\)' to '\\(i,\\)'. The process could then be repeated for \\(I,J,K,\\) etc.

It is easier to say \\(I\\) is a shape, so it could be \\(10,\\) but it could also be \\(10, 128, 128, 3.\\) Then rather than saying \\(i < I\\) to indicate the coordinate in \\(I\\), we say \\(i\in I\\).

We can take this a step further by letting \\(I,J\\) denote a partitioning of a shape. For example, in \\(N,S,C,\\) \\(N\\) could be the batch size, \\(S\\) could be the spatial dimensions, and \\(C\\) the number of channels. In \\(n,s,c\\) the coordinates are concatenated back together. Applying to \\(10,128,128,3\\), we would have \\(N=10\\), \\(S=128,128\\) and \\(C=3\\). If \\(n=2\\), \\(s=5,24\\) and \\(c=1\\) then \\(n,s,c\\) is the coordinate \\(2,5,24,1.\\)

We can even take this further by letting \\(I,J\\) be a partitioning that carries axis positions. For example, \\(I\\) could be the odd axes and \\(J\\) could be the even axes, and they could even match different axes in different tensors. And with the axes carrying the position, \\(I,J\\) and \\(J,I\\) mean the same thing. Automatic differentiation doesn't care about axis order or how tensors are implemented. On the other hand, it is very important when implementing the operations.

If \\(I\\) is empty, we treat it as having one empty coordinate, so \\(I,J\\) acts like \\(I\\) and the coordinates in \\(I,J\\) are the same as those in \\(J\\). This is the same as how a tensor with no axes is a scalar.

For coordinates outside of the shape, i.e. with an axis having a negative value or a value not less than the length in the shape for that axis, we assume the tensor is \\(0\\). This is not necessary, but it will simplify the description of some operations, such as convolution.

## Broadcast

In a broadcast we add dimensions that we ignore.
$$\begin{aligned}
y[I,J] &= \mathop{\mathtt{broadcast}}(x[I], J)\\\\
y[i,j] &= x[i].
\end{aligned}$$
This is an example where the fully generalized coordinates apply. The \\(I\\) and \\(J\\) can apply to any number of axes which may be intermingled with each other in any order on the left side.

The derivative gives the forward propagation:
$$\begin{aligned}
dy[i,j] &= dx[i]\\\\
dy[I,J] &= dx.
\end{aligned}$$

For backwards propagation, each \\(dx[i]\\) is incremented once for each \\(j\in J\\) in \\(dy[i,j]\\), so we can gather all these sums together, as follows:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx[i]}&\pluseq \overline{dy[i,j]}\\\\
\overline{dx[i]}&\pluseq \sum_{j\in J} \overline{dy[i,j]}\\\\
\overline{dx[I]} &\pluseq \sum_{j\in J} \overline{dy[I,j]}
\end{aligned}$$

## Broadcast vector addition

When adding a bias, the bias vector is broadcast. We could explicitly broadcast the bias vector and then add, or combine the two steps:

For
$$
\begin{aligned}
y[N, I] &= u[N, I] + b[I]\\\\
y[n,i] &= u[n,i] + b[i],
\end{aligned}$$
we see that for forward propagation,
$$\begin{aligned}
dy[n,i] &= du[n,i] + db[i],\\\\
dy[N,I] &= du[N,I] + db[I].
\end{aligned}$$
For backward propagation, we propagate to \\(db[i]\\) for every \\(dy[i,n]\\) so there will be a summation:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{du[N, I]} &\pluseq \overline{dy[N, I]}\\\\
\overline{db[I]}&\pluseq\sum_{n\in N} \overline{dy[n, I]}.
\end{aligned}$$

## Matrix multiplication by a vector

Matrix-vector multiplication is:
$$\begin{aligned}
y[I] &= w[I,J]v[J]\\\\
y[i] &= \sum_{j\in J} w[i,j]v[j].
\end{aligned}$$
Even with \\(J\\) a generalized coordinate, \\(v[J]\\) is still acting as a vector. If \\(J\\) were two-dimensional, this multiplication would be like in mnist-mlp when the 2d image is flattened and then multiplied by the weights.

The derivative is
$$\begin{aligned}
dy[i] &= \sum_{j\in J}\left(w[i,j]\\,dv[j]+dw[i,j]v[j]\right)\\\\
&=\sum_{j\in J}w[i,j]\\,dv[j] + \sum_{j\in J}dw[i,j]v[j].
\end{aligned}$$
Using the definition of matrix multiplication, for forward propagation,
$$dy[I] = w[I,J]\\,dv[J]+dw[I,J]v[J].$$

For backwards propagation,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dv[J]}&\pluseq \overline{dy[I]}w[I,J]\\\\
\overline{dw[i,j]}&\pluseq \overline{dy[i]}v[j].
\end{aligned}$$

## Matrix-matrix multiplication

$$\begin{aligned}
z[I,K] &= x[I,J]y[J,K]\\\\
z[i,k] &= \sum_{j\in J} x[i,j]y[j,k].
\end{aligned}$$
Although matrix multiplication is not commutative, and often not even defined if the order is changed, the generalized coordinates definition is commutative since the coordinates specify the axes. Also recall that order does not matter since \\(I, J,\\) and \\(K\\) carry the actual positions. The order in the definition is just to make things look like traditional multiplication.

If \\(K\\) (or \\(I\\)) is empty this definition reverts to the matrix-vector definition, and if \\(I\\) and \\(J\\) are empty it becomes the vector inner product. If \\(I\\) and \\(J\\) are empty, \\(x\\) is a scalar and the product becomes scalar-tensor multiplication. Finally, if \\(J\\) is empty the product becomes a "matrix" of the products of all vector element pairs.

The derivative gives the forward propagation, which looks like the scalar version:
$$\begin{aligned}
dz[i,k] &= \sum_{j\in J}\left(x[i,j]dy[j,k]+dx[i,j]y[j,k]\right)\\\\
&=\sum_{j\in J}x[i,j]dy[j,k] + \sum_{j\in J} dx[i,j]y[j,k]\\\\
dz[I,K] &= x[I,J]dy[J,K] + dx[I,J]y[J,K].
\end{aligned}$$

For the backwards propagation notice that each \\(dz[i,k]\\) uses a \\(dy[j,k]\\) (for all \\(i\\)) and a \\(dx[i,j]\\) (for all \\(k\\)) so
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dy[j,k]}&\pluseq \sum_{i\in I} \overline{dz[i,k]}x[i,j]\\\\
\overline{dx[i,j]}&\pluseq \sum_{k\in K} \overline{dz[i,k]}y[j,k]
\end{aligned}$$
These are matrix products, so:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dy[J,K]}&\pluseq \overline{dz[I,K]}x[I,J]\\\\
\overline{dx[I,J]}&\pluseq \overline{dz[I,K]}y[J,K].
\end{aligned}$$
Without the generalized coordinates there would be transposes to get the axes in the proper positions.

## Matrix-batch matrix multiplication
By adding batch axes, all the variants of matrix multiplication will be covered.

$$\begin{aligned}
z[N,I,K] &= x[I,J] y[N,J,K]\\\
z[n,i,k] &= \sum_{j\in J} x[i,j]y[n,j,k].
\end{aligned}$$

The derivative gives the forward propagation:
$$\begin{aligned}
dz[n,i,k]&= \sum_{j\in J}\left( x[i,j]dy[n,j,k] + dx[i,j]y[n,j,k] \right)\\\\
&=\sum_{j\in J}x[i,j]dy[n,j,k] + \sum_{j\in J}dx[i,j]y[n,j,k]\\\\
dz[N,I,K]&= x[I,J]dy[N,J,K] + dx[I,J]y[N,J,K].
\end{aligned}$$

For backward propagation, \\(dx[I,J]\\) requires summation on \\(N\\).
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dy[n,j,k]}&\pluseq \sum_{i\in I}\overline{dz[n,i,k]}x[i,j]\\\\
\overline{dx[i,j]}&\pluseq \sum_{n\in N}\sum_{k\in K} \overline{dz[n,i,k]}y[n,j,k]
\end{aligned}$$

These are matrix products, so
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dy[N,J,K]}&\pluseq \overline{dz[N,I,K]}x[I,J]\\\\
\overline{dx[I,J]}&\pluseq \overline{dz[I,(N,K)]}y[J,(N,K)],
\end{aligned}$$
where we combined the two summations in \\(dx\\) by merging the axes.

## Reductions

A reduction sums an axis of a tensor:
$$\begin{aligned}
y[I] &= \sum_{j\in J} x[I,j]\\\\
y[i] &= \sum_{j\in J} x[i,j].
\end{aligned}$$

The derivative gives the forward propagation:
$$\begin{aligned}
dy[i] &= \sum_{j\in J} dx[i,j]\\\\
dy[I] &= \sum_{j\in J} dx[I,j].
\end{aligned}$$
The forward propagation is a reduction, while the backwards propagation is a broadcast addition:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\overline{dx[I,J]}\pluseq \overline{dy[I]}.
$$

## Zero padding/Slice

Padding adds \\(0\\)s before and after the elements of some axes. Let the padded axes be \\(P\\), with \\(p_0, p_1\\) being padding to be added before and after the \\(P\\) axes in the input tensor.

$$\begin{aligned}
y[N,I=P+p_0+p_1]&=\mathop{\mathtt{pad}}(x[N,P], p_0, p_1)\\\\
y[n,i]&=\begin{cases}
x[n,i-p_0]&\text{if }p_0\le i < P+p_0\\\\
0&\text{otherwise}.
\end{cases}
\end{aligned}$$

Slicing removes elements outside of a region:
$$\begin{aligned}
y[N,I=S-s_0-s_1]&=\mathop{\mathtt{slice}}(x[N,S], s_0, s_1)\\\\
y[n,i]&=x[n,i+s_0]
\end{aligned}$$

The forward propagation for pad comes from the derivative:
$$\begin{aligned}
dy[n,i]&=\begin{cases}
dx[n,i-p_0]&\text{if }p_0\le p < P+p_0\text{ for all axes}\\\\
0&\text{otherwise}
\end{cases}\\\\
dy[n,I]&=\mathop{\mathtt{pad}}(dx[n,P], p_0, p_1).
\end{aligned}$$

The backward propagation for pad just needs to shift the coordinates:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx[n,p]}&\pluseq dy[n,p-p_0]\\\\
\overline{dx[N,P]}&\pluseq \mathop{\mathtt{slice}}(\overline{dy[N,I]}, p_0, p_1).
\end{aligned}
$$

For slice, the forward propagation is:
$$
\begin{aligned}
dy[n,i]&= dx[n,i+s_0]\\\\
dy[N,I]&= \mathop{\mathtt{slice}}(dx[N,S],s_0,s_1)
\end{aligned}
$$
and the backwards propagation is:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx[n,i+s_0]}&\pluseq \overline{dy[n,i]}\\\\
\overline{dx[N,S]}&\pluseq \mathop{\mathtt{pad}}(\overline{dy[N,I]},s_0,s_1).
\end{aligned}$$

## Simple dilation and striding

Dilation interleaves elements of the input tensor with a zero background. The dilation \\(d\\) is the spacing of elements from the input tensor, so each element has \\(d-1\\) \\(0\\)s between it and the next element on each axis.

$$\begin{aligned}
y[N,I=d(D-1)+1] &= \mathop{\mathtt{dilate}}(x[N,D], d)\\\\
y[n,i] &= \begin{cases}
x\left[n,\left\lfloor\frac{p}{d}\right\rfloor\right]&\text{if }p \bmod d = 0\text{ for all axes}\\\\
0&\text{otherwise}.
\end{cases}
\end{aligned}$$

Strides are the opposite of dilation; they select every \\(s\\)th element from the input.
$$\begin{aligned}
y\left[N,I=\left\lceil \frac{S}{s}\right\rceil\right] &= \mathop{\mathtt{stride}}(x[N,S], s)\\\\
y[n,i] &= x[n,is].
\end{aligned}$$

For forward propagation of dilation,
$$\begin{aligned}
dy[n,i]&=\begin{cases}
dx\left[n,\left\lfloor\frac{p}{d}\right\rfloor\right]&\text{if }p \bmod d = 0\text{ for all axes}\\\\
0&\text{otherwise}.
\end{cases}\\\\
dy[N,I]&=\mathop{\mathtt{dilate}}(dx[N,D],d).
\end{aligned}$$

For backward propagation of dilation,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx[n,i]}&\pluseq\overline{dy[n,di]}\\\\
\overline{dx[N,I]}&\pluseq\mathop{\mathtt{stride}}(\overline{dy[N,I]},d).
\end{aligned}
$$

For forward propagation of stride,
$$\begin{aligned}
dy[n,i]&=dx[n,is]\\\\
dy[N,I]&=\mathop{\mathtt{stride}}(dx[N,S],s).
\end{aligned}$$

For backward propagation of stride,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx[n,is]}&\pluseq \overline{dy[n,i]}\\\\
\overline{dx[N,S]}&\pluseq \mathop{\mathtt{dilate}}(\overline{dy[N,I]}, s).
\end{aligned}$$
The first form is more efficient since it avoids adding many \\(0\\)s.

## Slice with stride, dilation with pad

Slice is often combined with a stride. Dilation isn't combined with a pad, but it will be useful to do so later.

$$
\begin{aligned}
\mathop{\mathtt{slice}}(x[N,S],s_0,s_1,s_2) &= \mathop{\mathtt{stride}}(\mathop{\mathtt{slice}}(x[N,S],s_0, s_1),s_2)\\\\
\mathop{\mathtt{dilate}}(x[N,D],d,p_0,p_1)&= \mathop{\mathtt{pad}}(\mathop{\mathtt{dilate}}(x[N,D], d), p_0, p_1).
\end{aligned}
$$

## Basic convolution

The input \\(x[N,S_x,C_x]\\) of a convolution has spatial axes \\(S_x\\) and channel axes \\(C_x\\). A kernel \\(k[S_k,C_x,C_y]\\) has spatial axes \\(S_k\\), input channels \\(C_x\\) and output channels \\(C_y\\). The number of axes in \\(S_x\\) and \\(S_k\\) must be the same, and the length of each axis in \\(S_k\\) must be no greater than the corresponding axis in \\(S_x\\). The kernel is positioned over every location in the input with overlap (extended beyond the shape) and produces \\(C_y\\) channel outputs that are a linear combination of all the \\(C_x\\) channel values in the input covered by the kernel.

$$\begin{aligned}
y[N,S_y=S_x+S_k-1, C_y]&=\mathop{\mathtt{conv}}(x[N,S_x,C_x], k[S_k, C_x, C_y])\\\\
y[n,s_y,c_y]&=\sum_{s_k \in S_k}\sum_{c_x\in C_x} x[n,s_y+s_k-S_k+1,c_x]k[s_k,c_x,c_y].
\end{aligned}$$

The derivative gives the forward propagation:
$$
\begin{aligned}
dy[n,s_y,c_y]&=\sum_{s_k\in S_k}\sum_{c_x\in C_x}\left( x[n,s_y+s_k-S_k+1,c_x]dk[s_k,c_x,c_y]+dx[n,s_y+s_k-S_k+1,c_x]k[s_k,c_x,c_y]\right)\\\\
&=\sum_{s_k\in S_k}\sum_{c_x\in C_x}\left( x[n,s_y+s_k-S_k+1,c_x]dk[s_k,c_x,c_y]\right)+\sum_{s_k\in S_k}\sum_{c_x\in C_x}\left(dx[n,s_y+s_k-S_k+1,c_x]k[s_k,c_x,c_y]\right)\\\\
dy[N,S_y,C_y]&=\mathop{\mathtt{conv}}(x[N,S_x,C_x], dk[S_k,C_x,C_y])+\mathop{\mathtt{conv}}(dx[N,S_x,C_x], k[S_k,C_x,C_y]).
\end{aligned}
$$

For backwards propagation we need to fix \\(dk[s_k,c_x,c_y]\\) and \\(dx[n,s_x,c_x].\\)

For \\(dk\\), \\(x[s_y+s_k, c_x]\\) will have \\(s_y\\) take on all values in \\(S_y,\\) so
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dk[s_k,c_x,c_y]}&\pluseq\sum_{n\in N}\sum_{s_y\in S_y} \overline{dy[n,s_y,c_y]}x[n,s_y+s_k-S_k+1,c_x].
\end{aligned}
$$

For \\(dx\\),
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx[n,s_x,c_x]}&\pluseq 
\sum_{s_k\in S_k}
\sum_{c_y\in C_y} \overline{dy[n,s_x-s_k+S_k-1,c_y]}k[s_k,c_x,c_y].
\end{aligned}
$$

## Grouped Convolution

In a grouped convolution \\(C_x\\) and \\(C_y\\) are split into \\(G\\) groups be reshaping. We can treat this as adding an axis \\(G\\) to the input, kernel, and output.

$$\begin{aligned}
y[N,S_y=S_x+S_k-1, G, C_y]&=\mathop{\mathtt{conv}}(x[N,S_x,G, C_x], k[S_k, G, C_x, C_y])\\\\
y[n,s_y,g, c_y]&=\sum_{s_k \in S_k}\sum_{c_x\in C_x} x[n,s_y+s_k-S_k+1,g, c_x]k[s_k,g, c_x,c_y].
\end{aligned}$$
Instead of the kernel having \\(S_k(GC_x)(GC_y)\\) elements, it will have \\(S_kC_xC_yG\\) elements and each of the \\(G\\) slices of the input can be convolved with the \\(G\\) slices of the kernel in parallel to produce \\(G\\) slices of output.

The derivative gives the forward propagation:
$$
\begin{aligned}
dy[n,s_y,g,c_y]&=\sum_{s_k\in S_k}\sum_{c_x\in C_x}\left( x[n,s_y+s_k-S_k+1,g,c_x]dk[s_k,g,c_x,c_y]+dx[n,s_y+s_k-S_k+1,g,c_x]k[s_k,g,c_x,c_y]\right)\\\\
&=\sum_{s_k\in S_k}\sum_{c_x\in C_x}\left( x[n,s_y+s_k-S_k+1,g,c_x]dk[s_k,g,c_x,c_y]\right)+\sum_{s_k\in S_k}\sum_{c_x\in C_x}\left(dx[n,s_y+s_k-S_k+1,g,c_x]k[s_k,g,c_x,c_y]\right)\\\\
dy[N,S_y,G,C_y]&=\mathop{\mathtt{conv}}(x[N,S_x,G,C_x], dk[S_k,G,C_x,C_y])+\mathop{\mathtt{conv}}(dx[N,S_x,G,C_x], k[S_k,G,C_x,C_y]).
\end{aligned}
$$

For backwards propagation we need to fix \\(dk[s_k,g,c_x,c_y]\\) and \\(dx[n,s_x,g,c_x].\\)

For \\(dk\\), \\(x[n,s_y+s_k,g,c_x]\\) will have \\(s_y\\) take on all values in \\(S_y,\\) so
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dk[s_k,g,c_x,c_y]}&\pluseq\sum_{n\in N}\sum_{s_y\in S_y} \overline{dy[n,s_y,g,c_y]}x[n,s_y+s_k-S_k+1,g,c_x].
\end{aligned}
$$

For \\(dx\\),
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
\overline{dx[n,s_x,g,c_x]}&\pluseq 
\sum_{s_k\in S_k}
\sum_{c_y\in C_y} \overline{dy[n,s_x-s_k+S_k-1,g,c_y]}k[s_k,g,c_x,c_y].
\end{aligned}
$$

## Convolution with the works

In the simple grouped convolution we treated the input and kernel as though they were spatially infinite but zero outside of their valid coordinates and sized the output to contain only those elements that could be non-zero. This avoids messiness in back propagation near the edges, but some changes are needed for the deep learning definition.

The most general convolution has batch, stride, dilation, pads and groups:
$$y[N,S_y,G,C_y]=\mathop{\mathtt{conv}}(x[N,Sx,G,Cx], k[S_k,G,C_x,C_y],d,p_0,p_1,s).$$

Dilation is applied to the kernel \\(k\\), giving it an effective spatial size of \\(d(S_k-1)+1.\\)

Padding is applied to the input \\(x\\), giving it an effective spatial size of \\(p_0+S_x+p_1.\\) The padded input is convolved with the dilated kernel.

The convolution output is then sliced so that only the portion of the output where the dilated kernel completely overlaps the padded input is kept. Thus, the slice begins at \\(d(S_k-1)\\) inclusive (overlap of 1), and ends at \\(p_0+S_x+p_1\\), exclusive, for a size of \\(p_0+S_x+p_1-d(S_k-1).\\)

The sliced output is strided by \\(s\\) to give a size of
$$S_y=\left\lceil\frac{p_0+S_x+p_1-d(S_k-1)}{s}\right\rceil.$$

Putting the pieces together,
$$\begin{aligned}
k_d&=\mathop{\mathtt{dilate}}(k,d)\\\\
x_p&=\mathop{\mathtt{pad}}(x,p_0, p_1)\\\\
y_c&=\mathop{\mathtt{conv}}(x_p,k_d)\\\\
y_s&=\mathop{\mathtt{slice}}(y_c, d(S_k-1), p_0+S_x+p_1)\\\\
y&=\mathop{\mathtt{stride}}(y_s,s).
\end{aligned}$$

### Computing padding

Rather than being explicitly specified, the padding is often specified as one of \\(\texttt{VALID},\\)
\\(\texttt{SAME\_UPPER}\\), or \\(\texttt{SAME\_LOWER},\\) which are used to compute \\(p_0\\) and \\(p_1\\) based on the other parameters.

For \\(\texttt{VALID},\\) \\(p_0=p_1=0.\\)

With the \\(\texttt{SAME}\\) paddings, we want the output size to be the *same* as would be obtained by striding the input,
$$S_y=\left\lceil\frac{S_x}{s}\right\rceil.$$
At the same time, we want to minize \\(p=p_0+p_1\\) so we do not dilute the information in the input with excess \\(0\\)s. Thus we want the minimal \\(p\\) where
$$ 
\left\lceil\frac{S_x}{s}\right\rceil=\left\lceil\frac{p+S_x-d(S_k-1)}{s}\right\rceil.
$$

The floor is easier to work with than the ceiling. We can use the identity
$$
\left\lfloor\frac{x}{s}\right\rfloor=\left\lfloor\frac{x-1+s}{s}\right\rfloor
$$
to rewrite as
$$ 
\left\lfloor\frac{S_x-1+s}{s}\right\rceil=\left\lfloor\frac{p+S_x-1-d(S_k-1)+s}{s}\right\rfloor.
$$
Now let
$$
q=\frac{(S_x-1)-(S_x-1)\%s}{s}.
$$
so
$$S_X-1=qs+(S_x-1)\%s$$
and \\(0\le (S_x-1)\%s < s.\\)

Substituting we get
$$
\begin{aligned}
\left\lfloor\frac{qs+(S_x-1)\%s+s}{s}\right\rfloor&=\left\lfloor\frac{p+qs+(S_x-1)\%s-d(S_k-1)+s}{s}\right\rfloor\\\\
\left\lfloor q+1+\frac{(S_x-1)\%s}{s}\right\rfloor&=\left\lfloor q+1+\frac{p+(S_x-1)\%s-d(S_k-1)}{s}\right\rfloor\\\\
q+1&=\left\lfloor q+1+\frac{p+(S_x-1)\%s-d(S_k-1)}{s}\right\rfloor.
\end{aligned}
$$
For this to be true we need
$$0 \le p+(S_x-1)\\%s-d(S_k-1) < s.$$
Since we are minimizing \\(p\\), we want
$$
\begin{aligned}
0&=p+(S_x-1)\\%s-d(S_k-1)\\\\
p&=d(S_k-1)-(S_x-1)\\%s.
\end{aligned}
$$
It would not make sense for the effective filter size to not cover the stride since input would be thrown away. But it would make even less sense to throw even more input away with negative padding, so the actual value used is
$$p_0+p_1=\max(0, d(S_k-1)-(S_x-1)\\%s).$$

Padding is evenly divided between \\(p_0\\) and \\(p_1\\), with the excess going to \\(p_0\\) for \\(\texttt{SAME\_LOWER}\\) and \\(p_1\\) for \\(\texttt{SAME\_UPPER}.\\)
