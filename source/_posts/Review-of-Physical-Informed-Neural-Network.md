---
title: Review of Physical Informed Neural Network
author: Ryan LI
toc: true
declare: true
index_img: /index/PINN.png
tags:
  - fluid dynamics
  - deep learning
  - paper reading
date: 2022-06-09 02:20:20
---

> It's a brief note of the talk: [Physics-Informed Deep learning](https://www.bilibili.com/video/BV19a41167RU?share_source=copy_web), by [Dr. Lu Lu](https://lu.seas.upenn.edu/people/), based on the review paper: Physics-informed machine learning[^1]. During PHD, Lu Lu worked with [Prof. George Em Karniadakis](https://www.brown.edu/research/projects/crunch/george-karniadakis) in the same group proposing the PINN[^2] and now he is an assistant professor of the Pennsylvania University. 

<!-- more -->

{% note primary %}

Here, we review some of the prevailing trends in embedding physics into machine learning, present some of the current capabilities and limitations and discuss diverse applications of physics-informed learning both for forward and inverse problems, including discovering hidden physics and tackling high-dimensional problems.

{% endnote %}

{% note secondary %}

Additional resouces:

Papers:

- [Physics Informed Deep Learning (Part I): Data-driven Solutions of Nonlinear Partial Differential Equations, M.Raissi, P.Perdikaris, G.E.Karniadakis](https://arxiv.org/abs/1711.10561) 

- [Physics Informed Deep Learning (Part II): Data-driven Discovery of Nonlinear Partial Differential Equations, M.Raissi, P.Perdikaris, G.E.Karniadakis](https://arxiv.org/abs/1711.10566)

Course and slides:

- [CS 598: Deep Generative and Dynamical Models](https://arindam.cs.illinois.edu/courses/f21cs598/)

- [CS598: Physics-Informed Neural Networks: A deep learning framework for solving forward and inverse problems involving nonlinear PDEs](https://arindam.cs.illinois.edu/courses/f21cs598/slides/pml11_598f21.pdf)
- [Physics-Informed Neural Networks (PINNs)](https://mltp2020.com/Presentations/Karniadakis_NSF_MLTP2020.pdf)

Blog:

- [DS1 Physics Informed Neural Networks](https://cs598ban.github.io/Fall2021/ds/physics+ml/2021/11/18/DS1_blog2.html)

{% endnote %}





[^1]: [Karniadakis, G. E., Kevrekidis, I. G., Lu, L., Perdikaris, P., Wang, S., & Yang, L. (2021). Physics-informed machine learning. *Nature Reviews Physics*, *3*(6), 422-440.](https://scholar.google.com/scholar_url?url=https://www.nature.com/articles/s42254-021-00314-5&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=12413463696550326945&ei=ku-gYuLyIuOEywThnbSYCQ&scisig=AAGBfm2hbAYVf-NUg8tveih4kCyCAE_8rA)
[^2]: [Raissi, M., Perdikaris, P., & Karniadakis, G. E. (2019). Physics-informed neural networks: A deep learning framework for solving forward and inverse problems involving nonlinear partial differential equations. *Journal of Computational physics*, *378*, 686-707.](https://www.sciencedirect.com/science/article/pii/S0021999118307125)
