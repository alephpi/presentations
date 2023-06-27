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
Today I'm going to present you a newly coming out paper which proposes new model architecture called RwaKuv, which reinvents RNN in transformer era. Although the paper has only been released recently, the development dates back to 2021 and it has already shown its competence in LLM field. The model is open-sourced on github and available on HF.
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
</div>

::right::
# Time-Mixing

<br>

$$
\begin{align*}
    r_t &= W_r \cdot (\mu_r x_t + (1 - \mu_r) x_{t-1} ) \\
    k_t &= W_k \cdot (\mu_k x_t + (1 - \mu_k) x_{t-1} ) \\
    v_t &= W_v \cdot (\mu_v x_t + (1 - \mu_v) x_{t-1} ) \\
    wkv_t &= \frac{ \sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} v_i + e^{u+k_t} v_t }{\sum_{i=1}^{t-1} e^{-(t-1-i)w+k_i} + e^{u+k_t}} \\
    o_t &= W_o \cdot (\sigma(r_t) \odot wkv_t)
\end{align*}
$$

- $wkv_t$ analogous to dot production attention
- RWKV stands for four trainable parameters: 
**R**eceptance, **W**eight, **K**ey, **V**alue

---

# $wkv$: RNN-Transformer Duality

## Time-parallel mode


---
layout: two-cols
---
# Compared to LSTM

<br>

<div class='-ml-30'>

$$
\begin{align*}
f_t &= \sigma_g(W_{f} x_t + U_{f} h_{t-1} + b_f) \\
i_t &= \sigma_g(W_{i} x_t + U_{i} h_{t-1} + b_i) \\
o_t &= \sigma_g(W_{o} x_t + U_{o} h_{t-1} + b_o) \\
\tilde{c}_t &= \sigma_c(W_{c} x_t + U_{c} h_{t-1} + b_c) \\
c_t &= f_t \odot c_{t-1} + i_t \odot \tilde{c}_t\\
h_t &= o_t \odot \sigma_h(c_t)
\end{align*}
$$
</div>

- four gates
- two hidden states
- lots of non-linearities

::right::
# Compared to Transformer



---

# Some other words

<br>

Bo Peng was almost an outsider of the game, who is neither a researcher in academic nor in industry.

As the authors claim, it is the first RNN and also non-Transformer architecture that scales up to 10B with competitive performance as Transformer.

The model is continuing to upscale with the help of open source community.<sup>*</sup> 

<Footnotes separator>
  <Footnote :number="'*'">Check news on <a href='https://discord.gg/bDSBUMeFpc'>Discord</a>!</Footnote>
</Footnotes>

Bo wishes the RWKV could replace Transformer as the basic architecture in LLM. He also believes AI development should be fully open sourced. He wants RWKV foundation becoming the **Linux** in LLM era.