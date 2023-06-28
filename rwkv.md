---
theme: academic
background: null
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: fade
title: RWKV
fonts:
  sans: ''
coverDate: null
---

# RWKV

## Reinventing RNN in Transformer Era

<div class='p-3'/>

### Peng Bo et al.

<div class='p-3'/>

_Presented by Sicheng_

<span>{{ new Date().toLocaleDateString() }}</span>

<!--
Today I'm going to present you a newly coming out paper which proposes new model architecture called RwaKuv, which reinvents RNN in transformer era. Although the paper has only been released recently, the development dates back to 2021 and it has already shown its potential competence in LLM field. The model is open-sourced on github and available on HF.
-->

---
layout: fact
---

## Transformer vs RNN

<br>
<br>

|    Model    | parallizable | scalable |    Time    |      Space      |
| :---------: | :----------: | :------: | :---------: | :-------------: |
| Transformer |      ✓      |    ✓    | $O(T^2d)$ | $O(T^2 + Td)$ |
|  RNN/LSTM  |      ×      |    ×    |  $O(Td)$  |    $O(d)$    |

<!--
Let's first give some motivations. We all know that Transformer and RNN are two main architectures in language modeling task, while transformer has recently dominated this area, becoming the base architecture especially for large language model design, since transformer is more parallizable and scalable compared to RNN/LSTM. Even though, Transformer suffers from the square complexity in both time and space during inference stage, while the RNN is better at inference with a linearly growing time and constant memory usage.
-->

---

## RWKV takes both!

<br>
<br>

|    Model    | parallizable | scalable |    Time    |      Space      |
| :---------: | :----------: | :------: | :---------: | :-------------: |
| Transformer |      ✓      |    ✓    | $O(T^2d)$ | $O(T^2 + Td)$ |
|  RNN/LSTM  |      ×      |    ×    |  $O(Td)$  |    $O(d)$    |
|    RWKV    |      ✓      |    ✓    |  $O(Td)$  |    $O(d)$    |

<br>

<div v-click class='text-red-500 text-10'>How?</div>

<!--
So RWKV tries and succeeds to inherit both of their advantages. Wow! that sounds cool! So how does it happen?
-->

---
layout: two-cols
---

<img src="/rwkv/rwkv-block.png"/>

::right::

# Architechture Overview

RWKV architecture is designed for [language modeling](https://github.com/BlinkDL/RWKV-LM) task.<sup>*</sup> 
<Footnotes separator>
  <Footnote :number="'*'">but not restricted to, there are already varieties in community works</Footnote>
</Footnotes>

As usual, the architecture is composed by:
- a beginning embedding part
- a main block: time mixing + channel mixing
- a language modeling head



<br>
<br>
<br>

RWKV stands for four trainable parameters: 
**R**eceptance, **W**eight, **K**ey, **V**alue

<!--
Let me introduce its architecture step by step. Here is an overview of RWKV architecture. Just as usual architecture designed for language modeling task, it begins with an  embedding layer and ends with a language modeling head. In between is the main block which again is composed of a time-mixing block and a channel-mixing block with common operations like layernorm and residual connections. Notice that the main block is stacked vertically to have multiple layers and also expanded horizontally as recurrence. One more thing before we go on, RWKV is named after its four important parameters, receptance, weight, key and value, we will see immediately what do they refer to.
-->

---
layout: two-cols
---

# Channel-Mixing

<br>

<div class='pr-30'>

$$
\begin{align*}
r_t &= W_r \cdot (\mu_r x_t + (1-\mu_r)x_{t-1}) \\
k_t &= W_k \cdot (\mu_k x_t + (1-\mu_k)x_{t-1}) \\
o_t &= \sigma(r_t) \odot (W_v \cdot \max(k_t, 0)^2)
\end{align*}
$$

- interpolation between current and last input 
- linear receptance $r$ and key $k$
- only the output gate with non-linearity
- $\mu$ hyperparameter 
</div>

::right::
<div v-click>

# Time-Mixing

<br>

<span class="text-gray mb-0 pb-0">

$$
\begin{align*}
    r_t &= W_r \cdot (\mu_r x_t + (1 - \mu_r) x_{t-1} ) \\
    k_t &= W_k \cdot (\mu_k x_t + (1 - \mu_k) x_{t-1} ) \\
\end{align*}
$$

</span>

$$
\begin{align*}
    v_t &= W_v \cdot (\mu_v x_t + (1 - \mu_v) x_{t-1} ) \\
    wkv_t &= \frac{ \sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} v_i + e^{u+k_t} v_t }{\sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} + e^{u+k_t}} \\
    o_t &= W_o \cdot (\sigma(r_t) \odot wkv_t)
\end{align*}
$$

- $wkv_t$ serves as the attention score
- $u$ hyperparamter for numerical stability
</div>

<!--
Now let's go into the channel-mixing block. There are several variables, t is the current timestep index, x is the input, r for receptance, k for key, o for output and W's are trainable matrices. 

Then the time-mixing block is a little bit more complicated, the .

The reason that we call it channel mixing and time-mixing is because here we .
-->

---

# $wkv$: A closer look

$wkv$ is the attention-like score inspired by [Attention Free Transformer](https://arxiv.org/abs/2105.14103)

<div grid="~ cols-2 gap-2" my4>

<div>

$$
wkv_t = \frac{ \sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} v_i + e^{u+k_t} v_t }{\sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} + e^{u+k_t}}
$$

</div>

<div>

$$
\begin{align*}
\mathrm{Attn}(Q, K, V)_t &= \frac{\sum_{i=1}^{T} e^{q^{\top}_t k_i}v_i}{\sum_{i=1}^{T} e^{q^{\top}_t k_i}}
\end{align*}
$$

</div>
</div>

🙂
- summation over all past timesteps (not need explicit causal mask)
- weight decay over time (not need explicit positional encoding)
<!-- - infinite context length -->
- computationally cheaper (scalar product and sum instead of outer product)
- still parallelizable (except for time-dimension)

🙁
- less expressive (sacrafice dot-product attention and query vector)

<!--
Now let's take a closer look at wkv score. it is inspired by AFT. We list the wkv and original self-attention here side by side. Both of them are sums weighted by some exponential distributions.
-->

---

# $wkv$: Duality

$$
wkv_t = \frac{ \sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} v_i + e^{u+k_t} v_t }{\sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} + e^{u+k_t}}
$$

We can rewrite it in recurrent form:

$$
\begin{align*}
    a_0, b_0 &= 0\\
    wkv_t &= \frac{a_{t-1} + e^{u + k_t} v_t}{b_{t-1} + e^{u + k_t}} \\
    a_t &= e^{-w} a_{t-1} + e^{k_t} v_t \\
    b_t &= e^{-w} b_{t-1} + e^{k_t}
\end{align*}
$$

<br>

We call them **time-parallel mode** and **time-sequential mode** respectively.

<!--
The most intriguing property of such wkv score is the duality, we can not only express it as a self-attention like score, but also if we rewrite it by these terms, it can be regarded as recurrent hidden states. We call them time-parallel mode and time-sequential mode respectively. In particular, we can make use the parallel mode to train the model just like a Transformer while we make use the sequential mode to inference like an RNN. That's why rwkv architecture can take advantages from both.
-->

---

<img src="/rwkv/inference-speed.png">

<!--
Finally some figures reporting the performance of the rwkv model. In inference stage, (linear vs quadratic).
-->

---

<img src="/rwkv/evaluations.png">

<!--
And this picture shows the competence as well as the scalability of rwkv model against other open-sourced gpt-like model on different language tasks. we can see that the orange curve(RWKV) is at the same level of the other competitors.
-->

---

# Useful resources

<br>

[RWKV paper](https://arxiv.org/abs/2305.13048)

[RWKV repo](https://github.com/BlinkDL/RWKV-LM)

[RWKV in 150 lines](https://github.dev/BlinkDL/ChatRWKV/blob/main/RWKV_in_150_lines.py)

[How the RWKV language model works](https://johanwind.github.io/2023/03/23/rwkv_details.html)(Blog)

[RWKV paper explained](https://youtu.be/x8pW19wKfXQ)(Youtube Video)
---

# Some other words

<br>


Bo Peng was almost an outsider of the game, who is neither a researcher in academic nor in industry.

As the authors claim, it is the first RNN and also non-Transformer architecture that scales up to 10B with competitive performance as Transformer.

The model is continuing to upscale with the help of open source community.<sup>*</sup> 

<Footnotes separator>
  <Footnote :number="'*'">Check news on <a href='https://discord.gg/bDSBUMeFpc'>Discord</a>!</Footnote>
</Footnotes>

Bo wishes the RWKV could replace Transformer as the basic architecture for LLM. He also believes AI development should be fully open sourced. He wants RWKV foundation becoming as the **Linux** in LLM era.

---
layout: cover
class: text-center
coverDate: null
---

# Thank you
