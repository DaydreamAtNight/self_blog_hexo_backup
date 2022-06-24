---
title: 'Introduction, Machine learning for fluids dynamics'
author: Ryan LI
toc: true
declare: true
index_img: /index/Lesson_note_ML_for_fluid_dynamics.jpeg
tags:
  - fluid dynamics
  - deep learning
date: 2022-06-01 17:09:11
---

> This is a series of brief notes for the popular lesson: [Machine Learning for Fluid Mechanics](https://www.youtube.com/watch?v=8e3OT2K99Kw&list=PLMrJAkhIeNNQWO3ESiccZmPssvUDFHL4M&index=1), by [Dr. Steve Brunton](https://www.eigensteve.com/). He is not only a good instructor, but an active researcher focusing combining techniques in dimensionality reduction, sparse sensing, and machine learning for the data-driven discovery and control of complex dynamical systems. 

<!-- more -->

{% note primary %}

Given the background of the popularity of ML in the CV area, this lesson focus on how to apply it into the field of traditional physics science and engineering, especially dynamical systems and fluid mechanics. 

{% endnote %}

## Introduction

Several Q&As:

- Explain machine learning in nutshell.

  Machine learning is a growing set of techniques for high dimensional, non-convex optimisations based on a growing wealth of data.

- Why machine learning suits fluid mechanics? 

  Almost all of the fluid dynamics tasks (including Reduction, modelling, control, sensing and closure) can be written as nonlinear, non-convex, multi-scale and very high dimensional (very hard) **optimisation problems** that can't be solved efficiently by traditional methods. Yet it is exactly the field of machine learning. 

- What is high dimensional?

  Fluid itself has many degrees of freedom, million or billion degrees of freedom might be needed for simulate a turbulent fluid. And a growing of data leads an explosion of dimension.

- What is non-linear?

  Because fluids is governed by a nonlinear PDE, the NS equation.

- What is non-convex?

  Because there exists local minima in the optimisation problems.

- What kind of ML do we need?

  Interpretable, generalisable i.e. reliable

  Sparse, low-dimensional, robust

## History

Machine learning and fluid dynamics share a long history of interfaces. 

Pioneered by Rechenberg (1973) and Schwefel (1977), who introduced Evolution Strategies (ES) to design and optimise the profile of a multi-panel plate in order to minimise the drag. The stochastic was introduced and a optimisation similar to the SGD is applied to find the best configuration. 

Another link is Sir James Lighthill's report (1974), which leads the AI winter. His report says AI fails to meet the promises in several fields, such as speech recognition, and natural language processing (NLP).

{% note success %}

The first time I heard this part of history, I thought this Lighthill was some idiot who lacks insight. I never connected this man with the very person who proposed the Lighthill's Equation with the acoustic analogy method and founded the subject of aeroacoustics, during his PHD!

Also, it reminds me a talk I saw by Geoffrey Hinton, who said he came to the US because he couldn't find a job in the UK with a degree in AI. Back then (1978) UK refused to fund almost any research topics related to AI and deep learning, partly of Sir Lighthill's consequences. 

It's just like someone of my field narrowly buried the AI halfway yet at the same time influenced a star in this area.

{% endnote %}

## Examples

### POD/PCA/SVA

POD refers to the PCA done on the flow data. With a series of snapshots of a flow past a cylinder at $Re\approx100$, subtracting the mean flow and applying a singular value decomposition (SVA), the resulting dominant eigen flows (modes, representing the flow patterns) can be used to construct a **reduced order model** to reproduce the fluid flow field efficiently.

<img src="POD Analysis of Cylinder Flow.jpeg" alt="POD Analysis of Cylinder Flow. (a) Original Flow Field (vorticity shown). (b) First 8 dominant POD modes. After https://www.researchgate.net/publication/318710028_Special_Topics_in_CFD/citations" style="zoom:50%;" />

One application of this method is shown below[^1], RPCA, a variant of PCA is applied to denoise the artificially pepper-salt corrupted DNS simulation result, and real PIV experiment result. The outliers and the true fluid field are able to separate with a change of the $\lambda$.

<img src="RPCA for denoising.png" alt="RPCA filtering removes noise and outliers in the flow past a cylinder (black circle), from DNS (left) with 10% of velocity field measurements corrupted with salt and pepper noise, and PIV measurements (right). All frames show resultant vorticity fields. As the parameter λ is decreased, RPCA filtering is more aggressive, eventually incorrectly identifying coherent flow structures as outliers. After Scherl, I., Strom, B., Shang, J. K., Williams, O., Polagye, B. L., & Brunton, S. L. (2020). Robust principal component analysis for modal decomposition of corrupt fluid flows. *Physical Review Fluids*, *5*(5), 054401." style="zoom:48%;" />

### Closure modelling

<p align = "center"><iframe width="560" height="315" src="https://www.youtube.com/embed/Wr984EOmNaY?start=15" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

This is a DNS simulation on turbulent boundary layers by HTL. As we zoom in and out, massive fractal, multi-scale patterns are found in space and time. This happens everytime and everywhere when fluid flowing past a wing, car, or inside an engine. But instead of simulating the full DNS equation i.e. resolving the patterns with all the energy scales, we only want to get a reduced order model approximating the small energy scales while focusing on only the scales of energy that resulting in the macroscopic change of the fluid.

{% note info %}

Steve reckons this as the one of the most exciting area where ML can really make a practical impact on everyday industrials. And I heard that after a talk in China, he said a great progress on the area of turbulence might be made in the next 2 decades. (I'm not sure, just heard of it)

{% endnote %}

<img src="RANs modeling with DNN.png" alt="The novel architecture used for Rans closure model. Unlike the traditional MLP, an extra Tenser Input Layer, fixed during training, is introduced before the emerging layer. After Ling, J., Kurzawski, A., & Templeton, J. (2016). Reynolds averaged turbulence modelling using deep neural networks with embedded invariance. Journal of Fluid Mechanics, 807, 155-166." style="zoom:50%;" />

One inspiring job[^2] is mentioned which study the RANs closure models. They designed a novel customised structure that embed the physical variables one layer before the output layer of NN. And it inspires us how to "bake in" prior knowledge to design a model not only accurate but also physically meaningful".

### Super resolution

Super resolution is already a mature field in image sciences, and it can be directly deployed into flow fields.

<img src="Super resolution.png" alt="Super resolution reconstruction for turbulence flow, the interpolation error of the SHALLOW DECODER error is about 9.3%. From Erichson, N. B., Mathelin, L., Yao, Z., Brunton, S. L., Mahoney, M. W., & Kutz, J. N. (2020). Shallow neural networks for fluid flow reconstruction with limited sensors. *Proceedings of the Royal Society A*, *476*(2238), 20200097." style="zoom:50%;" />

Above is the result of reconstructing the turbulence flow fields([Johns Hopkins Turbulence Database](http://turbulence.pha.jhu.edu/)) from the coarse results obtained by applying an average pooling on the original flow fields[^3]. Multiple MLPs are deployed for this task. 

<img src="super resolution interpolation and extrapolation comparison.png" alt="Two different training and test set configurations, showing (a) a within sample prediction task and (b) an out of sample prediction task. Here, the gray columns indicate snapshots used for training, while the red columns indicate snapshots used for testing. From Erichson, N. B., Mathelin, L., Yao, Z., Brunton, S. L., Mahoney, M. W., & Kutz, J. N. (2020). Shallow neural networks for fluid flow reconstruction with limited sensors. *Proceedings of the Royal Society A*, *476*(2238), 20200097." style="zoom:50%;" />

Yet the good results only happen at the interpolation scenario, for the extrapolation i.e. prediction task, the reconstruction failed. More physics need to add to make this work.

### Deep autoencoders

The classical POD/PCA can be written in a form of one-layer linear autoencoder. And instead solving by the analytic SVD algorithm, it can be solved by SGD. So why not change the two-layer, linear autoencoder into the non-linear, multilayer deep autoencoder?

<img src="Deep autoencoder reconstruction.png" alt="Deep autoencoder reconstruction examples, (a) NN performs good, (b) NN performs bad. From Milano, M., & Koumoutsakos, P. (2002). Neural network modeling for near wall turbulent flow. Journal of Computational Physics, 182(1), 1-26." style="zoom:48%;" />

Based on this thought, Michele Milano and Petros Koumoutsakos deployed neural network modeling for near wall turbulent flow[^4], and compared with POD results, 2 decades ahead of its time. 

<img src="parabolic reduced order model.png" alt="Transient solution of the Navier–Stokes equation (solid circles) and Galerkin model B (solid curve). The figure shows (a1(t), a?(t)) of a transient trajectory starting close to the steady Navier–Stokes solution corresponding to the fixed point in the Galerkin system. From Noack, B. R., Afanasiev, K., MORZYŃSKI, M., Tadmor, G., & Thiele, F. (2003). A hierarchy of low-dimensional models for the transient and post-transient cylinder wake. *Journal of Fluid Mechanics*, *497*, 335-363" style="zoom:50%;" />

This concepts are also developed to build the reduced order models. The pattens extracted from POD or AE can be used to build simple representations in a low dimensional coordinate. For example Bernd R. Noack et als' work[^5] showed the dynamics of transient cylinder wakes can be concluded by a hierarchy of low-dimensional models lived on parabolic bowls.

<img src="Schematic overview of the proposed sparse modeling procedure.png" alt="Schematic overview of the proposed sparse modeling procedure. A sparse dynamical system is identified based on features obtained from sensor signals s and the full state u may also be estimated with the ability of PIV snapshots (optional). From Loiseau, J. C., Noack, B. R., & Brunton, S. L. (2018). Sparse reduced-order modelling: sensor-based dynamics to full-state estimation. *Journal of Fluid Mechanics*, *844*, 459-490" style="zoom:50%;" />

And these learned coordinates can be used with data-diven methods like SINDy(the sparse identification of nonlinear dynamics)[^6] to build efficient models for predicting the modes purely from measurement data. Jean-Christophe Loiseau has done a lot work based on this[^7].

### The ultimate goal:  Flow control

It is a very principled optimisation of the flow field and of the control law with some objectives in mind. Those objectives come from the real wold such as: increasing the lift, decreasing the drag. And these optimisation problems can be solved better with machine learning tools. 

<img src="Structure of control scheme.png" alt="Structure of the control scheme, where a classical MPC controller based on a model for the full system state is shown in green and a controller using a surrogate model in orange. After. After ieker, K., Peitz, S., Brunton, S. L., Kutz, J. N., & Dellnitz, M. (2019). Deep model predictive control with online learning for complex physical systems. arXiv preprint arXiv:1905.10094." style="zoom:50%;" />

As shown by Bieker at als' diagram[^8], instead of controlling a NS equation, an efficient alternative is controlling  a machine learning surrogate model to realise the real-time control.

## Inspiration from biology

At last to give us some confidence, Steve mentioned without knowing the NS equations, eagle can somehow manoeuvre the complex turbulence flow by its own sensors on its wings. And the insects, with way small neural system,  they can handle the complex turbulence flow elegantly and seamlessly (I don't feel any self-confidence hearing this). And maybe we can learn something from them and fit into our engineering world. 



[^1]:[Scherl, I., Strom, B., Shang, J. K., Williams, O., Polagye, B. L., & Brunton, S. L. (2020). Robust principal component analysis for modal decomposition of corrupt fluid flows. *Physical Review Fluids*, *5*(5), 054401.](https://scholar.google.com/scholar_url?url=https://journals.aps.org/prfluids/abstract/10.1103/PhysRevFluids.5.054401&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=9368717229379747223&ei=eXCcYp2tDZb0yAT_6KLYDw&scisig=AAGBfm3dnQgKr4Ku82Fv4l8EQlBowAQZdQ)
[^2]:[Ling, J., Kurzawski, A., & Templeton, J. (2016). Reynolds averaged turbulence modelling using deep neural networks with embedded invariance. Journal of Fluid Mechanics, 807, 155-166.](https://scholar.google.com/scholar_url?url=https://www.cambridge.org/core/journals/journal-of-fluid-mechanics/article/reynolds-averaged-turbulence-modelling-using-deep-neural-networks-with-embedded-invariance/0B280EEE89C74A7BF651C422F8FBD1EB&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=17265256326087997648&ei=iXCcYrLaKuOEywThnbSYCQ&scisig=AAGBfm1mnR-uoBACuF1rcikuCwEx5HWzKw)
[^3]: [Erichson, N. B., Mathelin, L., Yao, Z., Brunton, S. L., Mahoney, M. W., & Kutz, J. N. (2020). Shallow neural networks for fluid flow reconstruction with limited sensors. *Proceedings of the Royal Society A*, *476*(2238), 20200097.](https://scholar.google.com/scholar_url?url=https://royalsocietypublishing.org/doi/abs/10.1098/rspa.2020.0097&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=4644643361595480852&ei=72-cYoPaH8nFywSZv6aYDQ&scisig=AAGBfm3cHzLDR-qONvANNbFZQE0NFSr01Q)
[^4]:[Milano, M., & Koumoutsakos, P. (2002). Neural network modeling for near wall turbulent flow. Journal of Computational Physics, 182(1), 1-26.](https://www.semanticscholar.org/paper/Neural-network-modeling-for-near-wall-turbulent-Milano-Koumoutsakos/f0e8198850dae31cc9612940581ec8c059c24475)
[^5]:[Noack, B. R., Afanasiev, K., MORZYŃSKI, M., Tadmor, G., & Thiele, F. (2003). A hierarchy of low-dimensional models for the transient and post-transient cylinder wake. *Journal of Fluid Mechanics*, *497*, 335-363.](https://www.cambridge.org/core/journals/journal-of-fluid-mechanics/article/hierarchy-of-lowdimensional-models-for-the-transient-and-posttransient-cylinder-wake/0F114BEB5DD20B7342E99ED8D0070C01)
[^6]:[Brunton, S. L., Proctor, J. L., & Kutz, J. N. (2016). Discovering governing equations from data by sparse identification of nonlinear dynamical systems. *Proceedings of the national academy of sciences*, *113*(15), 3932-3937.](https://www.pnas.org/content/113/15/3932.short)
[^7]:[Loiseau, J. C., Noack, B. R., & Brunton, S. L. (2018). Sparse reduced-order modelling: sensor-based dynamics to full-state estimation. *Journal of Fluid Mechanics*, *844*, 459-490.](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=4rvzkMEAAAAJ&citation_for_view=4rvzkMEAAAAJ:qxL8FJ1GzNcC)
[^8]:[Bieker, K., Peitz, S., Brunton, S. L., Kutz, J. N., & Dellnitz, M. (2019). Deep model predictive control with online learning for complex physical systems. *arXiv preprint arXiv:1905.10094*.](https://arxiv.org/abs/1905.10094)
