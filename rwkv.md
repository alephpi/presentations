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
  sans: ""
coverDate: null
---
# RWKV

## Reinventing RNN in Transformer Era

<div class='p-3'/>

### Peng Bo et al.

<div class='p-3'/>

_Presented by Sicheng_

<span>{{ new Date().toLocaleDateString() }}</span>

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

---
layout: two-cols
---

<img src="/rwkv/rwkv-block.png"/>

::right::

# Architechture Overview