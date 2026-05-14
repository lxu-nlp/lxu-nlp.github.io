---
layout: post
title: "Notes on ML Gradients"
date: 2026-05-14
categories: [Notes]
tags: [ML, gradient]
math: true
---

# LoRA Backward and Activation Caching: Backward-First Walkthrough

本文用一个通用的第 `k` 层 Transformer 来说明：在只对 `K/V` projection 做 LoRA finetuning、且不使用 activation checkpointing 时，backward 如何决定 forward 中哪些 activation 必须保留，哪些可以省掉。

核心原则：

> **A forward tensor must be kept if some later backward formula asks for it.**

换句话说，不是先问“这个 forward tensor 要不要缓存”，而是从 backward 公式反推：某个 backward step 需要哪个 forward tensor，那个 tensor 才必须在 forward 中保留。

---

## 1. Chain rule basics

### 1.1 Scalar chain rule

如果：

\[
y = f(x), \quad L = g(y)
\]

那么：

\[
\frac{dL}{dx}
=
\frac{dL}{dy}\frac{dy}{dx}
\]

也就是：**上游梯度 upstream gradient** 乘以当前函数的 **局部导数 local derivative**。

### 1.2 Multivariate chain rule

如果：

\[
y = f(x_1, x_2, \ldots, x_n), \quad L = g(y)
\]

那么每个输入变量都有自己的梯度：

\[
\frac{\partial L}{\partial x_i}
=
\frac{\partial L}{\partial y}
\frac{\partial y}{\partial x_i}
\]

如果 `x` 和 `y` 都是向量，严格写法会涉及 Jacobian。实际 deep learning 框架一般不会显式构造完整 Jacobian，而是直接计算 **vector-Jacobian product (VJP)**：给定上游梯度 `dy`，直接算出 `dx`。

> **Key insight: VJP is the operation that turns a module's output gradient into its input gradient.**

---

## 2. What a module backward computes

考虑一个通用 module：

\[
y = f(x, \theta)
\]

其中：

- `x` 是 module input activation；
- `theta` 是 module 参数；
- `y` 是 module output activation。

当 backward 到达这个 module 时，后面的 module 已经传来了：

\[
dy = \frac{\partial L}{\partial y}
\]

当前 module backward 通常会产生两类梯度：

\[
d\theta = \frac{\partial L}{\partial \theta}
\]

和：

\[
dx = \frac{\partial L}{\partial x}
\]

它们是两条不同目的的路径。

---

### 2.1 Parameter / weight gradient: update the current module

`dtheta` 用来更新当前 module 自己的参数。

如果 `theta` trainable，则需要：

\[
d\theta
\]

如果 `theta` frozen，则不需要。

例如 linear：

\[
y = xW
\]

如果 `W` trainable：

\[
dW = x^\top dy
\]

这个 `dW` 才会交给 optimizer。

所以 parameter-gradient path 的直觉是：

> **How should this module's own weights change?**

它通常需要当前 module forward 时的 input activation，例如 linear 的 `dW = x^T dy` 需要 `x`。

---

### 2.2 Input gradient: pass the error signal backward

`dx` 不是为了更新当前 module，而是为了把梯度继续传给更早的 module。

如果 `x` 前面还有 trainable 参数，就必须计算：

\[
dx = \frac{\partial L}{\partial x}
\]

否则梯度链断掉，前面的参数收不到学习信号。

对当前 module 来说，`dx` 是 input gradient；对前一个 module 来说，这个同一个 `dx` 就是它收到的 upstream gradient。

\[
\text{current module input gradient}
=
\text{previous module upstream gradient}
\]

所以 input-gradient path 的直觉是：

> **How much should earlier modules be blamed for the loss?**

它的目标不是更新当前 module，而是把 loss signal 往前传。

> **Key insight: parameter gradient updates the current module; input gradient carries the error signal to earlier modules. Frozen parameters remove parameter gradients, but they do not remove input gradients if earlier trainable modules still need the signal.**

更具体地说：

> **For the upstream-gradient / input-gradient path, as long as there are earlier trainable parameters, the required backward computation and its required cache are basically independent of whether the intermediate weights are frozen. Freezing mainly removes parameter-gradient computation and tensors used only for `dW`.**

---

### 2.3 What forward activation does input-gradient actually need?

这里要区分 linear 和 nonlinear。

对于 pure linear：

\[
y = xW
\]

input-gradient backward 是：

\[
dx = dy W^\top
\]

这一步主要用：

- upstream gradient `dy`；
- weight `W`。

它不需要原始 input activation `x`。

所以如果 `W` frozen，并且不算：

\[
dW = x^\top dy
\]

那么 `x` 不需要为了这个 frozen linear 被保存。

但对 nonlinear / norm / attention，input-gradient backward 往往需要 forward 中间量。例如：

- GELU / SiLU / SwiGLU backward 需要 pre-activation 或 activation output；
- LayerNorm / RMSNorm backward 需要 mean/variance/rms 或 normalized output；
- attention backward 需要 `Q,K,V`、softmax probabilities 或 FlashAttention stats。

> **Key insight: for frozen linear input-gradient, the original input activation is often not needed; for nonlinear, norm, and attention input-gradient, forward intermediates are often essential.**

这也是 LoRA activation saving 的核心 nuance：LoRA/frozen training 可以省掉一部分 frozen linear 的 `dW`-only cache，但不能省掉 input-gradient path 上 nonlinear/norm/attention 需要的 cache。

---

## 3. PyTorch autograd intuition

PyTorch autograd 的核心机制是：**forward 时动态构建计算图，backward 时按反向拓扑顺序执行每个 op 的 backward rule。**

### 3.1 Forward builds a dynamic graph

当某个 tensor 的 `requires_grad=True`，并且你对它做 differentiable operation 时，PyTorch 会为这个 operation 创建一个 `grad_fn` 节点。

这个节点记录两类信息：

1. 这个 op 的 backward 规则是什么；
2. backward 需要哪些 forward tensors / stats。

这些需要留到 backward 的 forward tensors 通常叫 **saved tensors**。

例如：

- linear 如果需要 `dW`，backward 需要 input `x`；
- GELU backward 需要 input 或等价 activation；
- LayerNorm backward 需要 mean/variance/rms 等 stats；
- attention backward 需要 `Q,K,V`、softmax stats 或 FlashAttention stats。

> **Key insight: activation cache is not an arbitrary list chosen by the user; it is mostly the set of tensors that autograd nodes saved because their backward formulas will need them.**

### 3.2 Backward executes local backward rules

当你调用：

```python
loss.backward()
```

PyTorch 从 loss 开始，初始梯度是：

\[
\frac{\partial L}{\partial L} = 1
\]

然后沿计算图反向传播。每个 node 收到自己的 upstream gradient，执行本地 backward rule，返回它每个输入的 gradient。

对一个 op：

\[
y = f(x, \theta)
\]

它的 backward 可能返回：

\[
dx, d\theta
\]

其中：

- `dtheta` 会累积到 leaf parameter 的 `.grad`，供 optimizer 使用；
- `dx` 会继续传给产生 `x` 的前一个 node。

非 leaf tensor 的 gradient 通常只是中间传递值，不会默认保存在 `.grad` 里，除非显式调用 `retain_grad()`。

### 3.3 Frozen weights in autograd

如果某个 weight 设置了：

```python
W.requires_grad_(False)
```

那么 PyTorch 不会为它累积：

\[
dW
\]

也不会为 optimizer 保存它的 optimizer states。

但是，如果这个 op 的 input 仍然需要 gradient，例如前面还有 LoRA / embedding / trainable module，那么这个 op 仍然必须参与计算：

\[
dx
\]

因此 frozen weight 不等于 no backward。

> **Key insight: in PyTorch terms, `requires_grad=False` on a weight disables that weight's `.grad`, but it does not automatically detach the computation graph through the module if its input still requires grad.**

### 3.4 `no_grad` / `detach` is different from frozen weights

如果你用：

```python
with torch.no_grad():
    y = module(x)
```

或者：

```python
y = module(x).detach()
```

那是直接切断计算图。此时 backward 不会穿过这个 module，前面的 trainable 参数也收不到梯度。

所以：

- frozen weights: 不更新当前 weight，但可以继续传 input gradient；
- `no_grad` / `detach`: 切断 graph，不再传 input gradient。

这对 LoRA 很关键：如果低层 LoRA 还要训练，上层 frozen Transformer block 不能简单 `no_grad`，否则梯度传不回低层 LoRA。

### 3.5 Backward implementation intuition

从实现角度看，一个 autograd op 的 backward 可以理解成一个函数：

```python
def backward(ctx, grad_output):
    saved = ctx.saved_tensors
    grad_input = ...
    grad_weight = ...
    return grad_input, grad_weight
```

其中：

- `grad_output` 就是这个 op 收到的 upstream gradient；
- `ctx.saved_tensors` 是 forward 为 backward 保存的 tensors；
- 返回值要和 forward 的输入一一对应；
- 不需要梯度的输入可以返回 `None`。

所以 activation cache 的本质是：forward 时 `save_for_backward(...)` 保存了 backward 公式将来会用到的东西。

如果某个 weight frozen，autograd 可以不返回 / 不累积这个 weight 的 gradient；但如果 input 需要 gradient，backward 仍然要返回 `grad_input`。

---

## 4. A simple MLP example: upstream gradient and input gradient

先看一个没有 LoRA 的两层 MLP：

\[
h_1 = xW_1
\]

\[
a_1 = \phi(h_1)
\]

\[
y = a_1W_2
\]

\[
L = loss(y)
\]

假设 `W2` frozen，但 `W1` trainable。Backward 从 loss 开始：

\[
dy = \frac{\partial L}{\partial y}
\]

### Step 1: Backward through frozen `W2`

\[
y = a_1W_2
\]

因为 `W2` frozen，不需要：

\[
dW_2 = a_1^\top dy
\]

但为了更新前面的 `W1`，必须继续传梯度：

\[
da_1 = dy W_2^\top
\]

这里的 `da1` 是 activation `a1` 的 input gradient，也是前一个 operation `phi` 收到的 upstream gradient。

这一步计算 `da1` 只需要 `dy` 和 frozen `W2`，不需要 `a1`。所以 `a1` 不必为了 `dW2` 被保存。

### Step 2: Backward through activation

\[
a_1 = \phi(h_1)
\]

为了得到：

\[
dh_1
\]

通常需要 forward 的 `h1` 或 activation output，取决于 `phi`。

例如 GELU/SwiGLU 这类非线性，input-gradient backward 需要 forward 中间量。

因此，即使后面的 `W2` frozen，只要还要把梯度传到 `W1`，activation backward 所需 cache 仍然要保留。

### Step 3: Backward through trainable `W1`

\[
h_1 = xW_1
\]

现在 `dh1` 是这个 linear 的 upstream gradient。

因为 `W1` trainable，需要：

\[
dW_1 = x^\top dh_1
\]

如果还要继续往更早层传，则还需要：

\[
dx = dh_1 W_1^\top
\]

这个例子说明：

\[
dy \rightarrow da_1 \rightarrow dh_1 \rightarrow dW_1
\]

其中 `W2` 是否 frozen 只影响 `dW2` 是否计算；只要 `W1` 还要训练，`dy -> da1 -> dh1` 这条 input-gradient chain 仍然必须存在。

> **Key insight: freezing a downstream weight does not stop gradient propagation through it. It only stops computing that weight's own parameter gradient.**

---

## 5. LoRA backward in one linear projection

LoRA projection 写成：

\[
y = xW_0 + s xAB
\]

其中：

- `W0` frozen；
- `A, B` trainable；
- `s = alpha / r`。

定义 bottleneck：

\[
z = xA
\]

则：

\[
y = xW_0 + s zB
\]

给定上游梯度：

\[
dy = \frac{\partial L}{\partial y}
\]

LoRA backward 需要计算 LoRA 参数梯度：

\[
dB = s z^\top dy
\]

所以 backward 需要 forward 的：

\[
z = xA
\]

又因为：

\[
dA = s x^\top dy B^\top
\]

所以 backward 需要 forward 的：

\[
x
\]

同时，为了把梯度传给更早的 module，还需要 input gradient：

\[
dx = dy W_0^\top + s dy B^\top A^\top
\]

这个 `dx` 不是为了更新 `W0`，而是为了让更早层的 LoRA / embedding / trainable module 收到梯度。

在后面的第 `k` 层 walkthrough 中，它对应这条链：

\[
dK_k,dV_k
\rightarrow
 dx_k^K,dx_k^V
\rightarrow
 dx_k
\rightarrow
 dh_{k-1}
\rightarrow
 \text{lower-layer LoRA gradients}
\]

> **Key insight: in LoRA finetuning, `dx` is the bridge that lets an upper layer's loss signal reach lower-layer LoRA adapters. Without input gradients through frozen blocks, only the topmost trainable modules could learn.**

对 frozen base weight：

\[
dW_0 = x^\top dy
\]

不需要计算。

因此，`x` 不再因为 `dW0` 被保存；但是如果这个 projection 上挂了 LoRA，`x` 仍然因为 `dA` 必须保存。

> **Key insight: in a LoRA-targeted projection, the large input activation `x` is no longer needed for frozen `dW0`, but it is still needed for LoRA `dA`. So the main activation `x` is often not eliminated.**

---

## 6. Generic layer `k`: K/V LoRA only

设第 `k` 层 attention 输入为：

\[
h_{k-1}
\]

Attention norm 后：

\[
x_k = LN^{attn}_k(h_{k-1})
\]

Q 是 frozen projection：

\[
Q_k = x_k W^Q_k
\]

K/V 是 LoRA projection：

\[
K_k = x_k W^K_{0,k} + s x_k A^K_k B^K_k
\]

\[
V_k = x_k W^V_{0,k} + s x_k A^V_k B^V_k
\]

定义：

\[
Z^K_k = x_k A^K_k
\]

\[
Z^V_k = x_k A^V_k
\]

Attention：

\[
S_k = Q_k K_k^\top / \sqrt{d_h}
\]

\[
P_k = softmax(S_k)
\]

\[
C_k = P_k V_k
\]

Output projection frozen：

\[
O_k = C_k W^O_k
\]

然后经过 residual、MLP，得到：

\[
h_k
\]

下面从 backward 开始推。

---

## 7. Backward-first walkthrough for layer `k`

假设从第 `k+1` 层或 loss 传来了：

\[
dh_k
\]

目标是：

1. 计算第 `k` 层 K/V LoRA 参数梯度；
2. 产生 `dh_{k-1}`，继续传给第 `k-1` 层。

---

### Step 1: Final residual

如果：

\[
h_k = u_k + f_k
\]

Backward：

\[
du_k \mathrel{+}= dh_k
\]

\[
df_k = dh_k
\]

Residual add 只是分流梯度，不需要额外大 activation。

---

### Step 2: Frozen MLP

MLP 参数 frozen，但 backward 仍要从：

\[
df_k
\]

得到：

\[
dm_k
\]

因为还要继续往前产生：

\[
dh_{k-1}
\]

如果 MLP 是 SwiGLU：

\[
a_k = m_k W^{up}_k
\]

\[
g_k = m_k W^{gate}_k
\]

\[
r_k = silu(g_k) \odot a_k
\]

\[
f_k = r_k W^{down}_k
\]

Backward 中：

- `silu` backward 需要 `g_k` 或等价 activation；
- elementwise product backward 需要 `a_k` 和 `silu(g_k)`；
- frozen linear input-gradient 需要 frozen weights，但不需要 input for `dW`。

因此反推出：

- forward 需要保留 MLP nonlinearity backward 所需中间量；
- forward 不需要保留只为 frozen MLP weight gradient 服务的 input cache。

> **Key insight: frozen MLP still needs activation for input-gradient through nonlinearities, but not for weight-gradient of frozen linear weights.**

---

### Step 3: MLP norm

若：

\[
m_k = LN^{mlp}_k(u_k)
\]

Backward 要从：

\[
dm_k
\]

得到：

\[
du_k^{mlp}
\]

LayerNorm / RMSNorm 的 input-gradient 依赖 forward stats，例如 mean/variance/rms 或 normalized output。

因此反推出：

- forward 需要保留 MLP norm stats / input 等价信息；
- 即使 norm 参数 frozen，也需要这些信息来算 input gradient。

---

### Step 4: Attention residual

若：

\[
u_k = h_{k-1} + O_k
\]

Backward：

\[
dh_{k-1}^{residual} \mathrel{+}= du_k
\]

\[
dO_k = du_k
\]

Residual add 不需要额外 cache。

---

### Step 5: Frozen output projection

\[
O_k = C_k W^O_k
\]

Backward 为了继续传梯度，只需要：

\[
dC_k = dO_k (W^O_k)^\top
\]

这一步不需要 forward 的 `C_k`。

只有要算：

\[
dW^O_k = C_k^\top dO_k
\]

才需要 `C_k`。

但 `W^O_k` frozen，所以不算 `dW^O_k`。

因此反推出：

- forward 不需要为了 output projection 的 weight gradient 保留 `C_k`；
- 但如果 attention backward 或具体 kernel 另有需要，那是其他原因。

---

### Step 6: Attention output

\[
C_k = P_k V_k
\]

Backward：

\[
dP_k = dC_k V_k^\top
\]

\[
dV_k^{attn} = P_k^\top dC_k
\]

因此反推出：

- 为了算 `dP_k`，forward 需要 `V_k`；
- 为了算 `dV_k`，forward 需要 `P_k` 或等价 stats。

如果是 naive attention，可能保留完整：

\[
P_k \in \mathbb{R}^{B \times n_h \times T \times T}
\]

如果是 FlashAttention，则通常保留 compact stats，并在 backward 中重算局部 attention probabilities。

---

### Step 7: Softmax

\[
P_k = softmax(S_k)
\]

Softmax backward 需要 `P_k` 或等价统计量。

因此反推出：

- forward 需要保留 softmax backward 信息；
- 这部分和参数是否 frozen 没有直接关系。

---

### Step 8: Attention score

\[
S_k = Q_k K_k^\top / \sqrt{d_h}
\]

Backward：

\[
dQ_k = dS_k K_k / \sqrt{d_h}
\]

\[
dK_k^{attn} = dS_k^\top Q_k / \sqrt{d_h}
\]

因此反推出：

- 为了算 `dQ_k`，forward 需要 `K_k`；
- 为了算 `dK_k`，forward 需要 `Q_k`。

所以 attention backward 要保留 `Q_k, K_k`，或保留足够信息用于重算。

---

### Step 9: Frozen Q projection

\[
Q_k = x_k W^Q_k
\]

Backward：

\[
dx_k^Q = dQ_k (W^Q_k)^\top
\]

因为 `W^Q_k` frozen，不算：

\[
dW^Q_k = x_k^\top dQ_k
\]

因此反推出：

- Q projection 的 input-gradient 不需要 `x_k`；
- Q projection 的 weight-gradient 才需要 `x_k`；
- 但 `W^Q_k` frozen，所以不需要为 Q 的 `dW` 保存 `x_k`。

注意：这不代表 `x_k` 可以整体丢掉，因为 K/V LoRA 会需要同一个 `x_k`。

---

### Step 10: K-LoRA

\[
K_k = x_k W^K_{0,k} + s Z^K_k B^K_k
\]

\[
Z^K_k = x_k A^K_k
\]

从 attention backward 得到：

\[
dK_k
\]

#### 10.1 Gradient for `B_K`

\[
dB^K_k = s (Z^K_k)^\top dK_k
\]

这个公式需要：

\[
Z^K_k
\]

因此反推出：

- forward 必须保留 `Z^K_k = x_k A^K_k`，或者 backward 时重算；
- 无 checkpointing 下通常保留。

#### 10.2 Gradient for `A_K`

先有：

\[
dZ^K_k = s dK_k (B^K_k)^\top
\]

又因为：

\[
Z^K_k = x_k A^K_k
\]

所以：

\[
dA^K_k = x_k^\top dZ^K_k
\]

即：

\[
dA^K_k = s x_k^\top dK_k (B^K_k)^\top
\]

这个公式需要：

\[
x_k
\]

因此反推出：

- forward 必须保留 `x_k`；
- 这是 LoRA target projection 上大 activation 省不掉的原因。

#### 10.3 Input gradient through K path

\[
dx_k^K = dK_k (W^K_{0,k})^\top + dZ^K_k (A^K_k)^\top
\]

这里需要的是参数：

\[
W^K_{0,k}, A^K_k, B^K_k
\]

不额外需要 forward activation。

---

### Step 11: V-LoRA

\[
V_k = x_k W^V_{0,k} + s Z^V_k B^V_k
\]

\[
Z^V_k = x_k A^V_k
\]

从 attention backward 得到：

\[
dV_k
\]

#### 11.1 Gradient for `B_V`

\[
dB^V_k = s (Z^V_k)^\top dV_k
\]

需要：

\[
Z^V_k
\]

因此反推出：

- forward 必须保留 `Z^V_k`。

#### 11.2 Gradient for `A_V`

\[
dA^V_k = s x_k^\top dV_k (B^V_k)^\top
\]

需要：

\[
x_k
\]

因此反推出：

- forward 必须保留 `x_k`。

#### 11.3 Input gradient through V path

\[
dx_k^V = dV_k (W^V_{0,k})^\top + s dV_k (B^V_k)^\top (A^V_k)^\top
\]

需要的是参数，不额外需要 forward activation。

---

### Step 12: Merge attention input gradients

现在有三路：

\[
dx_k = dx_k^Q + dx_k^K + dx_k^V
\]

这个 `dx_k` 是 attention norm 的 upstream gradient。

也就是说，projection block 的 input gradient，变成了前一个 operation 的 upstream gradient。

---

### Step 13: Attention norm

\[
x_k = LN^{attn}_k(h_{k-1})
\]

Backward 要从：

\[
dx_k
\]

得到：

\[
dh_{k-1}^{attn}
\]

LayerNorm input-gradient 需要 forward stats / input 等价信息。

因此反推出：

- forward 必须保留 attention norm stats / input 等价信息；
- 这不是为了更新 norm 参数，而是为了得到 `dh_{k-1}`。

最终：

\[
dh_{k-1}
=
 dh_{k-1}^{residual}
+
 dh_{k-1}^{attn}
\]

第 `k` 层 backward 完成。这个 `dh_{k-1}` 会成为第 `k-1` 层 backward 的 upstream gradient。

---

## 8. What is kept, derived from backward

### 7.1 Kept for LoRA parameter gradients

K-LoRA：

\[
dA^K_k = s x_k^\top dK_k (B^K_k)^\top
\]

requires:

\[
x_k
\]

\[
dB^K_k = s (Z^K_k)^\top dK_k
\]

requires:

\[
Z^K_k
\]

V-LoRA 同理，需要：

\[
x_k, Z^V_k
\]

因此 K/V LoRA 至少要求 forward 保留：

\[
x_k, Z^K_k, Z^V_k
\]

### 7.2 Kept for attention input gradients

Attention backward 需要：

\[
V_k, P_k, Q_k, K_k
\]

或 FlashAttention-style compact stats + recomputation。

这些是为了从 `dC_k` 得到：

\[
dQ_k,dK_k,dV_k
\]

再继续传到 LoRA 和更低层。

### 7.3 Kept for norm / nonlinearity input gradients

LayerNorm / RMSNorm backward 需要 stats。

GELU / SwiGLU backward 需要 pre-activation 或等价 activation。

这些不是为了更新 frozen 参数，而是为了继续传：

\[
dh_k \rightarrow dh_{k-1}
\]

### 7.4 Can be skipped

Frozen linear 的 weight-gradient input cache 可以省。

例如：

\[
dW^Q_k = x_k^\top dQ_k
\]

不需要算，因为 `W_Q` frozen。

\[
dW^O_k = C_k^\top dO_k
\]

不需要算，因为 `W_O` frozen。

MLP frozen linear 的 weight gradients 也不需要。

但如果同一个 activation 被 LoRA gradient、attention backward、norm backward 需要，那它仍然必须保留。

> **Key insight: LoRA saves activation only where the only reason to keep a tensor was frozen weight-gradient computation. If the tensor is needed for LoRA gradients or input-gradient propagation, it cannot be dropped.**

---

## 9. Summary

从 backward 推导，规则只有一个：

> **A forward tensor must be kept if some backward formula later asks for it.**

Parameter-gradient path 和 input-gradient path 的区别：

- parameter gradient answers: **how should this module's own weights change?**
- input gradient answers: **how should the error signal be passed to earlier modules?**

在 K/V LoRA 中：

- `dA_K, dA_V` 要 `x_k`，所以 `x_k` 必须保留；
- `dB_K, dB_V` 要 `Z_K, Z_V`，所以 bottleneck 必须保留；
- attention backward 要 `Q,K,V,P/stats`，所以这些或等价信息必须保留；
- norm / nonlinearity backward 要 stats / pre-activation，所以也必须保留；
- frozen `Q/O/MLP/base K/base V` 不算 `dW`，所以只为这些 `dW` 服务的 input cache 可以省。

因此，LoRA 的主要显存节省来自：

- 不存 full `dW`；
- 不存 full optimizer states；
- 少部分 frozen weight-gradient-only activation cache。

而不是让 long-sequence activation 变成 inference-like。
