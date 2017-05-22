---
title: backpropagation
tags:
---

$\LaTeX Math$ 

Backpropagation:  compute the error vectors backwards

- goals of backpopagation
  - calculate $\frac{\partial C}{\partial w} $ and $\frac{\partial C}{\partial b}$
- two assumptions
  - the cost function can be written as an average $C = \frac{1}{n} \Sigma _x C_x $ 
    - reason: what backpropagation actually lets us do is compute the partial derivatives for a single training example
  - it can be written as a function of the outputs from the neural network $C = C(a^L)$ 
- fundamental equations
  - $\delta _j ^i \equiv \frac{\partial C}{\partial z_j^l}$ : the *error* in the $j^{th}$ neuron in the $l^{th}$ layer
  - $\delta = ((w ^ {l+1} \delta ^ {l+1} \bigodot \sigma'(z^l)))$
  - $\frac{\partial C}{\partial b} = \delta$
  - $\frac{\partial C}{\partial w_{jk}^l} = a_k^{l-1}\delta_j^l$
- Backpropagation Algorithm
  - input
  - feedforword
  - error
  - backpropagation
  - output

