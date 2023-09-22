---
title: "Automatic Differentiation"
date: 2023-09-19T14:53:04-07:00
math: true
---
# Introduction

When most people hear the term "Automatic differentiation," they think it is the same process learned in calculus only performed by a program. Others think it is a numeric approximation to the actual derivative. In reality, it is a clever transformation that extends a program that computes a function into one that also computes its derivatives with respect to specified parameters. Here we will provide a brief review of calculus followed by an explanation of automatic differentiation and some applications to deep learning computations.

# Computing derivatives

We'll start with finding a simple derivative three ways:
 - Symbolically, the way you learn in calculus,
 - Using forward propagation. This was the original way automatic differentiation was performed.
 - Using back propagation, the way deep learning frameworks do it. Backwards propagation tends to be more efficient when there is one computed value and many parameters.

The equation we'll use is:
$$f(x)=(x+a)(x+b).$$
This is a function of $x$ with parameters $a$ and $b.$ we want to find $\frac{df}{da}$ and $\frac{df}{db}$ for a fixed $x$.

## Symbolic

Symbolically,
$$\begin{align*}
df&=d((x+a)(x+a))\\\\
&=(x+a)\\,d(x+b) + (x+b)\\,d(x+a)\\\\
&=(x+a)\\,db + (x+b)\\,da.
\end{align*}$$

## Forward propagation

We begin by rewriting the computation $y=f(x)$ as a series of steps, each performing a primitive operation:
```python
t0 = load('x')  # x
t1 = load('a')  # a
t2 = t0 + t1    # x + a
t3 = load('b')  # b
t4 = t0 + t3    # x + b
t5 = t2 * t4    # (x + a) * (x + b)
y = t5
```
After each step, we also compute the derivative of the intermediate result with respect to $a$ and $b$ as relevant. We use `dt2_da` for $\frac{dt_2}{da}.$ To save space, for values such as $\frac{dt_0}{da}$ that are $0$ we omit `dt0_da` assuming it is $0$ in any use.

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
$\frac{dy}{da} = x + b$ and $\frac{dy}{db} = x + a.$

## Backward propagation

As with forward propagation we begin by rewriting the computation $y=f(x)$ as a series of primitive steps:
```python
t0 = load('x')  # x
t1 = load('a')  # a
t2 = t0 + t1    # x + a
t3 = load('b')  # b
t4 = t0 + t3    # x + b
t5 = t2 * t4    # (x + a) * (x + b)
y = t5
```
Now, for every `t_i` we add a `dt_i = 0:`
```python
dt0 = 0
dt1 = 0
dt2 = 0
dt3 = 0
dt4 = 0
dt5 = 0
```
Next we let `dy = 1`:
```python
dy = 1
```
Now we work backwards through the original computation looking for the first operation we haven't done a backprop for yet; in this case it is `y = t5.` We take the derivative in differential form: `dy = dt5`. We want it in the form of the sum of multiples of `dti`, which is already the case. Then for each such multiple, we had the product of the value on the left and the multiple to the `dt` variable:
```python
dt5 += dy  # 1
```
Now we move to the next unprocessed operation, `t5 = t2 * t4.` Taking the derivative gives `dt5 = t2 * dt4 + t4 * dt2` so we get two steps:
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
So $\frac{dy}{da}=x+b$ and $\frac{dy}{db}=x+a.$

If you think it is a bit mysterious how backwards propagation ends up with the correct values, your intuition is correct. If is often explained as just being the chain rule, along with some vigorous hand waving. Of course the chain rule is involved, but it is more interesting than just the chain rule. A future page will explain how it works.

# Extending to tensors

For a scalar $t$, we treat $dt$ as a scalar. For a tensor $t$ we treat $dt$ as a tensor with the same shape. But first we need to determine how to compute the derivative of the basic tensor operations.

## Vector addition

For a size $I$,
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
du[I] &\pluseq dy[I]\\\\
dv[I] &\pluseq dy[I].
\end{aligned}$$

### Broadcast vector addition

When adding a bias, the bias vector is broadcast.

For
$$y[N, I] = u[N, I] + b[I]$$
defined by
$$y[n,i] = u[n,i] + b[i],$$
we see that
$$dy[n,i] = du[n,i] + db[i],$$
so for forward propagation,
$$dy[N,I] = du[N,I] + db[I].$$
For backward propagation, we propagate to $db[i]$ for every $dy[i,n]$ so there will be a summation:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
du[N, I] &\pluseq dy[N, I]\\\\
db[I]&\pluseq\sum_{n<N} dy[n, I].
\end{aligned}$$

## Matrix multiplication

### By a vector

The matrix multiplication
$$y[I] = w[I,J]v[J],$$
is defined as
$$y[i] = \sum_{j<J} w[i,j]v[j].$$

Then
$$\begin{aligned}
dy[i] &= \sum_{j<J}\left(w[i,j]\\,dv[j]+dw[i,j]v[j]\right)\\\\
&=\sum_{j<J}w[i,j]\\,dv[j] + \sum_{j<J}dw[i,j]v[j].
\end{aligned}$$
Using the definition of matrix multiplication, for forward propagation,
$$dy[I] = w[I,J]\\,dv[J]+dw[I,J]v[J].$$

For backwards propagation,
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
dv[J]&\pluseq dy[I]w[I,J]\\\\
dw[i,j]&\pluseq dy[i]v[j].
\end{aligned}$$

### By a batch of vectors

We can easily extend this to multiplication by a batch of vectors, where:
$$y[N,I] = w[I,J]v[N,J],$$
is defined as
$$y[n,i] = \sum_{j<J} w[i,j]v[n,j].$$

Then
$$
\begin{aligned}
dy[n,i] &= \sum_{j<J}\left(w[i,j]\\,dv[n,j]+dw[i,j]v[n,j]\right)\\\\
&=\sum_{j<J}w[i,j]\\,dv[n,j] + \sum_{j<J}dw[i,j]v[n,j].
\end{aligned}$$
So for forward propagation,
$$dy[N,I] = w[I,J]\\,dv[N,J]+dw[I,J]v[N,J].$$

Like with batched vector addition, we again have summations for the terms without $N$:
$$\newcommand{\pluseq}{\mathrel{{+}{=}}}
\begin{aligned}
dv[N,J]&\pluseq dy[N,I]w[I,J]\\\\
dw[i,j]&\pluseq\sum_{n<N}dy[n,i]v[n,j].
\end{aligned}$$

## Reductions

A reduction sums an axis of a tensor:
$$y[I] = \sum_{j<J} x[I,j]$$
is defined by
$$y[i] = \sum_{j<J} x[i,j].$$

$$dy[i] = \sum_{j<J} dx[i,j].$$

In this case, the sum shows up in forward propagation:
$$dy[I] = \sum_{j<J} dx[I,j],$$

while backwards propagation is broadcast addition:
$$
\newcommand{\pluseq}{\mathrel{{+}{=}}}
dx[I,J]\pluseq dy[I].
$$
