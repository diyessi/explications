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

For
$$Y[I] = U[I] + V[I]$$
defined by
$$Y[i] = U[i] + V[i],$$
we see that
$$dY[i] = dU[i] + dV[i],$$
so
$$dY[I] = dU[I] + dV[I].$$

### Batched vector addition

When adding a bias, the bias vector is broadcast.

For
$$Y[I,N] = U[I,N] + B[I]$$
defined by
$$Y[i,n] = U[i,n] + B[i],$$
we see that
$$dY[i,n] = dU[i,n] + dB[i],$$
so
$$dY[I,N] = dU[I,N] + dB[I].$$

We will see that this requires careful treatment with back propagation.

## Matrix multiplication

### By a vector

The matrix multiplication
$$Y[I] = W[I,J]V[J],$$
is defined as
$$Y[i] = \sum_{j<J} W[i,j]V[j].$$

Then
$$\begin{aligned}
dY[i] &= \sum_{j<J}\left(W[i,j]\\,dV[j]+dW[i,j]V[j]\right)\\\\
&=\sum_{j<J}W[i,j]\\,dV[j] + \sum_{j<J}dW[i,j]V[j].
\end{aligned}$$
Using the definition of matrix multiplication,
$$dY[I] = W[I,J]\\,dV[J]+dW[I,J]V[J].$$

Conveniently, this turns into a sum of matrix-vector multiplications.

### By a batch of vectors

We can easily extend this to multiplication by a batch of vectors, where:
$$Y[I,N] = W[I,J]V[J,N],$$
is defined as
$$Y[i,n] = \sum_{j<J} W[i,j]V[j,n].$$

Then
$$\begin{aligned}
dY[i,n] &= \sum_{j<J}\left(W[i,j]\\,dV[j,n]+dW[i,j]V[j,n]\right)\\\\
&=\sum_{j<J}W[i,j]\\,dV[j,n] + \sum_{j<J}dW[i,j]V[j,n].
\end{aligned}$$
Using the definition of matrix multiplication by a batch of vectors,
$$dY[I,N] = W[I,J]\\,dV[J,N]+dW[I,J]V[J,N].$$

Like the batched vector addition, this requires care during backwards propagation.
