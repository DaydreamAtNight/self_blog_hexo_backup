---
title: 'Patterns, Machine learning for fluids dynamics'
author: Ryan LI
toc: true
declare: true
index_img: /index/Lesson_note_MLFD_patterns.jpeg
tags:
  - fluid dynamics
  - deep learning
date: 2022-06-05 17:42:04
---

> [The second course](https://www.youtube.com/watch?v=3fOXIbycAmc&list=PLMrJAkhIeNNQWO3ESiccZmPssvUDFHL4M&index=2&ab_channel=SteveBrunton) introduces the patterns and coherent structures in high-dimensional fluid dynamics and how machine learning is currently being used to extract them.

<!-- more -->

> This is a series of brief notes for the popular lesson: [Machine Learning for Fluid Mechanics](https://www.youtube.com/watch?v=8e3OT2K99Kw&list=PLMrJAkhIeNNQWO3ESiccZmPssvUDFHL4M&index=1), by [Dr. Steve Brunton](https://www.eigensteve.com/). He is not only a good instructor, but an active researcher focusing combining techniques in dimensionality reduction, sparse sensing, and machine learning for the data-driven discovery and control of complex dynamical systems. 

{% note primary %}

As we all know, computer vision is one major and advanced field of Machine learning. And the developed CV techniques can be leveraged directly to process fluid fields just by seeing them as images or movies. Some notable works as follows.

{% endnote %}

### Patterns exist

<img src="The-von-Karman-vortex-street-generated-by-the-Rishiri-island.jpeg" alt="The von Kármán vortex street generated by the Rishiri island of Hokkaido, Japan (top, photo from NASA, 2001; STS-100). This wake produced at high Reynolds number shares great similarity with the cylinder wake at low Reynolds number (bottom). After https://www.researchgate.net/publication/331768849_Modal_Analysis_of_Fluid_Flows_Applications_and_Outlook/figures?lo=1" style="zoom:50%;" />

This is the fundamental fact, even in the most complex systems, patterns exist. Just like there are dominant patterns (normally called latent features in the ML world) to define whether there is a human face or a dog in an image, there are dominant patterers to define a fluid field. 

{% note info %}

Interesting facts: In 1987, Sirovich wrote two papers that pioneered in two fields. In April, he applied the PCA/SVD algorithm to human faces to generate the "eigenfaces" for face recognition[^1]. Later in October, he applied this same technique into fluid fields to extract the coherent structures of flow fields[^2].

{% endnote %}

### POD/PCA and Autoencoder

#### Background

<img src="flow past a cylinder result.gif" alt="Flow past a cylinder result. After https://courses.ansys.com/index.php/courses/simple-approximations-of-fluid-flows/lessons/simulation-examples-homework-quizzes/topic/unsteady-flow-over-a-cylinder-simulation-example/" style="zoom:80%;" />

**POD:** Given a complex fluid field sequence such as the von Kármán vortex street, one can tell there's a simple regular pattern emerging here even if it has lots of pixels or generated by a sophisticate simulation with large degree of freedom. The patterns can be extracted by simple tools in linear algebra. For example, subtracting off the mean flow then deploying a singular vector decomposition to get a POD expansion as:
$$
\mathbf{u} \approx \bar{\mathbf{u}} + \sum^r_{k=1}\boldsymbol{\psi}_k(x)\mathbf{a}_k(t)
$$
It writes the spatial-temporal flow field as the mean flow plus the summation of several static eigenflow fields. And the eigen vector $\mathbf{a}$ changes with time enabling the summation also a function of time $t$.

POD method has been developed for 50 years and it is a cornerstone on how to analysis complex flow fields. 

{% note info %}

Note that this can be seen as a special form of the Fourier decomposition, space-time separation of variables. Fourier transform are very useful to decompose space-time variables and POD is a data-driven generalisation of the Fourier transform that satisfies the  particular fluid boundary conditions and is generated form physical data of an actual flow simulation.

{% endnote %}

<img src="PCA as auto-encoder.png" alt="PCA architecture as a one hidden layer, linear auto-encoder. From https://www.jeremyjordan.me/autoencoders/" style="zoom:50%;" />

**Autoencoder:** In this modern era, the POD/PCA can be rewritten in the form of the neural network as shown above. It works as a bottom neck information filter where the encoder compress the complex data into a latent space and the decoder reconstructs the full flow field image. And the objective is to minimise the distance between the reconstructed image and the original image. And by constraining the hidden layer size as much as possible, the encoder is able to distill the most important fluid coherent structure for reconstruction of the flow field image. 

<img src="deeper auto-encoder.png" alt="deeper auto-encoder architecture with multiple hidden layers and non-linear activation. From https://www.jeremyjordan.me/autoencoders/" style="zoom:50%;" />

**Deep Autoencoder:** Now a deep autoencoder with more hidden layers with non-linear activation functions can be deployed to enhance the performance i.e. smaller latent space in the middle, better coordinate representations of the flow field, and simpler representations to work with downstream tasks.

#### Example

<img src="Deep autoencoder reconstruction.png" alt="Deep autoencoder reconstruction examples, (a) NN performs good, (b) NN performs bad. From Milano, M., & Koumoutsakos, P. [3]" style="zoom:48%;" />

Michele Milano and Petros Koumoutsakos are the first to introduce AE into fluid dynamic. They applied neural network modelling for near wall turbulent flow[^3], and compared with POD results, 2 decades ahead of its time. 

### Robust PCA

Following the above work,  lots of things can be done such as robustify the extraction of patterns, on noisy data, corrupted data or data with outliers. 

#### Background

<img src="Removing shadows, specularities and saturations from face images.png" alt="Removing shadows, specularities, and saturations from face images. (a) Cropped and aligned images of a person’s face under different illuminations from the Extended Yale B database. The size of each image is 192 × 168 pixels, a total of 58 different illuminations were used for each person. (b) Low-rank approximationˆL recovered by convex programming. (c) Sparse errorˆS corresponding to specularities in the eyes, shadows around the nose region, or brightness saturations on the face. Notice in the bottom left that the sparse term also compensates for errors in image acquisition. After Candès et.al[6]" style="zoom:50%;" />

**PIV:** Particle Image Velocimetry (PIV) is an experimental technique to measure the fluid non-invasively. And the flow field tend to become highly noisy with higher speed and larger window. 

**RPCA:** While in the field of the image science, Candès et.al[^6] suggests the principal components of data can be recovered even if part of the data is arbitrarily corrupted. They describes the corrupted data as a superposition of a **Low rank component $L_0$** and a **sparse component $S_0$**, and the robust PCA is presented to recover each components. They also deploy the RPCA to recover the main characters from the background in surveillance videos and remove the shadows and specialities in faces images (as shown above). Why not apply it into the fluid flow images?

{% note success %} 

<img src="Cross_Correlation_Animation.gif" alt="Animated illustration of Cross Correlation algorithm. After https://commons.wikimedia.org/wiki/File:Cross_Correlation_Animation.gif" style="zoom:80%;" />

PIV uses the **cross-correlation algorithm**[^4] to determine the displacement of each sub-window. This is the exact same algorithm used in the CNNs. However, they call it "**convolution**"[^5], regardless of the fact that the actual convolution method is the transposition of the cross-correlation algorithm.

{% endnote %}

#### Example: 

<img src="RPCA for denoising.png" alt="RPCA filtering removes noise and outliers in the flow past a cylinder (black circle), from DNS (left) with 10% of velocity field measurements corrupted with salt and pepper noise, and PIV measurements (right). All frames show resultant vorticity fields. As the parameter λ is decreased, RPCA filtering is more aggressive, eventually incorrectly identifying coherent flow structures as outliers. After Scherl et.al [7]" style="zoom:48%;" />

Isabel Scherl et.al[^7] apply the RPCA algorithm to recover the the salt pepper corrupted flow fields, by solving a ralated relaxed optimisation problem. The low rank and sparse component refer to the coherent structure and the noise. And the POD and DMD modes separated from the recovered data can be highly optimised as well.

### Super resolution

#### Background

Super resolution is already a mature field in image sciences, and it can be directly deployed into flow fields.

#### Example

<img src="Super resolution.png" alt="Super resolution reconstruction for turbulence flow, the interpolation error of the SHALLOW DECODER error is about 9.3%. After Erichson, N. B. et.al [8]" style="zoom:50%;" />

Above is the result of reconstructing the turbulence flow fields([Johns Hopkins Turbulence Database](http://turbulence.pha.jhu.edu/)) from the coarse results obtained by applying an average pooling on the original flow fields[^3]. Multiple MLPs are deployed for this task. 

<img src="super resolution interpolation and extrapolation comparison.png" alt="Two different training and test set configurations, showing (a) a within sample prediction task and (b) an out of sample prediction task. Here, the gray columns indicate snapshots used for training, while the red columns indicate snapshots used for testing. After Erichson, N. B. et.al [8]" style="zoom:50%;" />

Yet the good results only happen at the interpolation scenario, for the extrapolation i.e. prediction task, the reconstruction failed. More physics need to add to make this work.

{% note info %}

Compared with the flow field prediction, all the CV tasks with large pretrained models are interpolation tasks. The training data already contains all the data that needed.

{% endnote %}

### Statistical stationarity

In stead of a simple flow passed a cylinder, most fluid fields in the real life are more complicated. It brings more difficulties for models to reconstruct the fluid.

#### Example

<img src="statistical stationarity.png" alt="Singular value spectra for the flows studied. The singular values for vortex shedding past a cylinder (blue) converge quickly, whereas the Gulf of Mexico vorticity data (purple) has a long tail. The sea surface temperature (yellow) and mixing layer vorticity (red) are of intermediate complexity. After Callaham, J. L et.al[9]" style="zoom:28%;" />

Callaham, J. L et.al [^9] apply robust flow reconstruction via sparse representation on flow pass a cylinder, mixing layer, sea surface temperature and gulf of Mexico. And the modes needed to reconstruct each flow fields increase as shown above.

This article mainly shows the sparse model outperforms the general model. But in the discussion session, it brings up the key requirements of the reconstruction: **sufficient training data** and **sufficient measured information**. And they quantify the rate of sufficiency in each cases.

<img src="sufficient training data.png" alt="Comparison of the amounts of training data needed to predict the test data. After Callaham, J. L et.al[9] " style="zoom:50%;" />

The residuals of projecting test data onto the linear subspaces of POD modes of increasing training data is provided. As more data is added to the training set, test set are more likely to be generalised by the training data modes. 

{% note danger %}

Personally, I don't really understand the term **projection**. Whether it is same as **reconstruction** but in an opposite direction? Need more knowledge on it.

{% endnote %}

For flow pass a cylinder, the model performs well even with very few training data since the flow is simple and periodic. However, the mixing layer and Gulf of Mexico vorticity data have relatively large residual, indicating that there are still new structures that haven’t been observed in the training data.

<img src="sufficient measurements.png" alt="Comparison of the amounts of measurements needed to reconstruct the test data. After Callaham, J. L et.al[9]" style="zoom:50%;" />

Above compares the normalised residual error of sparse representation-based reconstructions with increasing number of random point measurements. Similar to the research on amount of the training data, more information is needed from measurements to reconstruct a more complicate flow field.

{% note info %}

The result might be better if a powerful reconstruction model is used such as the Deep Autoencoder. It is still an open area.

{% endnote %}

### Reference

[^1]: [Sirovich, L., & Kirby, M. (1987). Low-dimensional procedure for the characterization of human faces. *Josa a*, *4*(3), 519-524.](https://scholar.google.com/scholar_url?url=https://www.osapublishing.org/abstract.cfm%3Furi%3Djosaa-4-3-519&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=2157283586688201779&ei=iIGcYpXMOP6J6rQPzPSs-A8&scisig=AAGBfm1UtAAHt6JMQKL_6kZNl8eYHaRD7g)
[^2]:[Sirovich, L. (1987). Turbulence and the dynamics of coherent structures. I. Coherent structures. *Quarterly of applied mathematics*, *45*(3), 561-571.](https://scholar.google.com/scholar_url?url=https://www.ams.org/qam/1987-45-03/S0033-569X-1987-0910462-6/&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=16097515379918329562&ei=r4GcYrGXCZb0yAT_6KLYDw&scisig=AAGBfm2FAMjBm8d7M0kmOWq1CQ5iribTeg)
[^3]:[Milano, M., & Koumoutsakos, P. (2002). Neural network modeling for near wall turbulent flow. Journal of Computational Physics, 182(1), 1-26.](https://www.semanticscholar.org/paper/Neural-network-modeling-for-near-wall-turbulent-Milano-Koumoutsakos/f0e8198850dae31cc9612940581ec8c059c24475)
[^4]: [Keane, R. D., & Adrian, R. J. (1992). Theory of cross-correlation analysis of PIV images. Applied scientific research, 49(3), 191-215. ](https://link.springer.com/article/10.1007/BF00384623)
[^5]: [LeCun, Y., Bottou, L., Bengio, Y., & Haffner, P. (1998). Gradient-based learning applied to document recognition. *Proceedings of the IEEE*, *86*(11), 2278-2324.](https://ieeexplore.ieee.org/abstract/document/726791/)
[^6]: [Candès, E. J., Li, X., Ma, Y., & Wright, J. (2011). Robust principal component analysis?. *Journal of the ACM (JACM)*, *58*(3), 1-37.](https://dl.acm.org/doi/abs/10.1145/1970392.1970395)
[^7]: [Scherl, I., Strom, B., Shang, J. K., Williams, O., Polagye, B. L., & Brunton, S. L. (2020). Robust principal component analysis for modal decomposition of corrupt fluid flows. *Physical Review Fluids*, *5*(5), 054401.](https://scholar.google.com/scholar_url?url=https://journals.aps.org/prfluids/abstract/10.1103/PhysRevFluids.5.054401&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=9368717229379747223&ei=eXCcYp2tDZb0yAT_6KLYDw&scisig=AAGBfm3dnQgKr4Ku82Fv4l8EQlBowAQZdQ)
[^8]:[Erichson, N. B., Mathelin, L., Yao, Z., Brunton, S. L., Mahoney, M. W., & Kutz, J. N. (2020). Shallow neural networks for fluid flow reconstruction with limited sensors. *Proceedings of the Royal Society A*, *476*(2238), 20200097.](https://scholar.google.com/scholar_url?url=https://royalsocietypublishing.org/doi/abs/10.1098/rspa.2020.0097&hl=en&sa=T&oi=gsb&ct=res&cd=0&d=4644643361595480852&ei=gcigYuSJPIOM6rQPoeQk&scisig=AAGBfm3cHzLDR-qONvANNbFZQE0NFSr01Q)
[^9]: [Callaham, J. L., Maeda, K., & Brunton, S. L. (2019). Robust flow reconstruction from limited measurements via sparse representation. *Physical Review Fluids*, *4*(10), 103907.](https://journals.aps.org/prfluids/abstract/10.1103/PhysRevFluids.4.103907)
