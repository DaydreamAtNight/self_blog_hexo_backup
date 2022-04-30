---
title: 'paper reading: AlexNet'
author: daydreamatnight
toc: true
declare: true
date: 2022-04-07 22:25:50
tags:
  - paper reading
  - deep learning
---

> It has been 10 years since AlexNet has been brought out. It is one of the cornerstones of this surge of deep learning.

> This is a [series of paper reading notes](https://daydreamatnight.github.io/2022/04/02/paper-reading-start/), hopefully, to push me to read paper casually and to leave some record of what I've learned.

<!-- more -->

paper link: [ImageNet Classification with Deep Convolutional Neural Networks](https://papers.nips.cc/paper/2012/hash/c399862d3b9d6b76c8436e924a68c45b-Abstract.html)

useful link: https://www.bilibili.com/video/BV1ih411J7Kz

## Little history

It hasn't gotten much attention by the area of machine learning for the first 2-3 years since it got published, because this paper is written rather as a technical report than an academic paper. A good paper needs new thoughts for the model, or at least some explanations, while this paper only presented how they applied 3 tricks and how good their results are. However, there was no doubt an influential hit in the area of computer vision, which has a passion for refreshing the top list. And this influence spread to other areas gradually with deeper studies on it.

## Notes by sections

### 0. Absturct 

> To make training faster, we used non-saturating neurons and a very efficient GPU implementation of the convolution operation.

In addition to a brief introduction to the model, the use of GPU is also mentioned in the abstract. And works around GPU are mentioned all the time. It was really a tough engineering job from the perspective of the first writer.  But it is not important for an acdemic paper. Besides, since the emergence of CUDA in 2007, the application of GPU in the ML field in 2012 is not uncommon, and MATLAB is mainly used as a ML tool with a large number of GPU acceleration libraries.

> We also entered a variant of this model in the ILSVRC-2012 competition and achieved a winning top-5 test error rate of 15.3%, compared to 26.2% achieved by the second-best entry

At last, the result in the ILSVRC-2012 competition is as good as knocking the second to the ground and then showing off with a set of backflips. So personally it might look like a technical report, but it's still an outstanding paper and absolutly worth reading.

### 7. Discussion

In stead of conclusion, this paper leaves a discussion as the last section, which is unsual.

> For example, removing any of the middle layers results in a loss of about 2% for the top-1 performance of the network. So the depth really is important for achieving our results

The depth is important, but it is insufficient to be simply concluded from the degradation caused by removing one middle layer, ignoring other effects such as superparameter settings. And, considering only the conculsion, a more complete one might be, depth and width are both very important. The ratio of height and width matters.

> To simplify our experiments, we did not use any unsupervised pre-training even though we expect that it will help

Before AlexNet, it was common to warm up the NN with massive unlabelled images before the actual training i.e. use an unsupervised model as an initial. And the goal of the field of machine learning was to extract the features of data through large-scale unsupervised models. However, this sentence steered the entire field from unsupervised to supervised learning, which, according to the machine learning pioneers such as Hinton and LeCun, was a "wrong route". But with the rise of the pre-trained language models such as Bert, and the contrative learning model in CV field such as MoCo, the unsupervised route is gradually comming back to the foreground. 

<img src="unsupervise learning cake.png" alt="unsupervise learning cake" style="zoom:80%;" />

> Ultimately we would like to use very large and deep convolutional nets on video sequences where the temporal structure provides very helpful information that is missing or far less obvious in static images.

Actually video sequences are still a tough area beacause of the high computational comsumpution and the copyright issues.

### Key figure

<img src="alexnet results.png" alt="alexnet results" style="zoom:67%;" />

The right part is the most important result in this paper, though it isn't been discussed much in this paper. Actually it shows the last layer feature vectors perform really well in the semantic space i.e. deep neural network is very suitable to extract features from data.

### 1. Introduction

> To improve their performance, we can collect larger datasets, learn more powerful models, and use better techniques for preventing overfitting.

The paper leads one route of deep learning, which is, with large dataset and model, developing powerful regularization methods to prevent overfitting. However there is a new route, which is focusing on designing good architecture s.t. the overfitting won't happen with large model.

> Our network contains a number of new and unusual features which improve its performance and reduce its training time, which are detailed in Section 3.

> we used several effective techniques for preventing overfitting, which are described in Section 4.

These two are the innovative points. People can then follow their work later, which makes this paper a cornerstone.

### 2. Dataset

> We did not pre-process the images in any other way, except for subtracting the mean activity over the training set from each pixel. So we trained our network on the (centered) raw RGB values of the pixels.

There is one more point that is not emphasized. Previously, features of an image (such as SIFT) were always used as input instead of raw RGB values. Datasets such as ImageNet provided SIFT of their image set as well. The end-to-end nature is the selling point of a series of deep learning papers that follow.

### 3. Architecture

#### 3.1. ReLU

From a present point of view, ReLU is not that important for speeding up the training process. Other  activation functions still work. It's the simplicity of ReLU that makes it stick.

### 4. Reducing Overfitting

A metaphore of overfitting: In order to get a high score on an exam, you memorize all the answers to the exercises instead of understanding the question.

#### 4.1 Data Augmentation

> The second form of data augmentation consists of altering the intensities of the RGB channels in training images. Specifically, we perform PCA on the set of RGB pixel values throughout the ImageNet training set.

PCA is here use as a augmentation method which follow-up work don't follow. For example, in ResNet a standard color augmentation is used with no fancy methods. And nowadays, standard color augmentation wins.

#### 4.2 dropout

> There is, however, a very efficient version of model combination that only costs about a factor of two during training. The recently-introduced technique, called “dropout”

Here dropout is considered a light version of model ensembling, but later [study below](https://jmlr.org/papers/volume15/srivastava14a.old/srivastava14a.pdf) has shown that the effect of dropout is actually equivalent to weight decay/regularization, yet there is no specific weight decay method equivalent to it algorithmically.

>> one way to obtain some of the benefits of dropout without stochasticity is to marginalize the noise to obtain a regularizer that does the same thing as the dropout procedure, in expectation. We showed that for linear regression this regularizer is a modified form of L2 regularization. For more complicated models, it is not obvious how to obtain an equivalent regularizer. 

### 5. Details of learning

> we trained our models using stochastic gradient descent with a batch size of 128 examples, momentum of 0.9, and weight decay of 0.0005

momentum, weight decay with SGD has become a standard method afterwards.

> We initialized the weights in each layer from a zero-mean Gaussian distribution with standard deviation 0.01

(0, 0.01) is usually chosen as the initialization parameter pair in most standard-sized models. (0, 0.02) is in use even for large models like Bert.

> We trained the network for roughly 90 cycles through the training set of 1.2 million images, which took five to six days on two NVIDIA GTX 580 3GB GPUs.

Similar to what is happening now with training NLP, maybe it will drive the next evolution in hardware. And probably hardware similar to TPU would be popular.

### 6. Results

#### 6.1 Qualitative Evaluations

> The kernels on GPU 1 are largely color-agnostic, while the kernels on on GPU 2 are largely color-specific. The kernels on GPU 1 are largely color-agnostic, while the kernels on on GPU 2 are largely color-specific

Interesting problem but less focused by follow up work.

> consider the feature activations induced by an image at the last, 4096-dimensional hidden layer. If two images produce feature activation vectors with a small Euclidean separation, we can say that the higher levels of the neural network consider them to be similar.

This is an intuitive work as talked before, and follow up work such as [Visualizing and understanding convolutional networks ](https://link.springer.com/chapter/10.1007/978-3-319-10590-1_53) dig deeper trying to interperate the NN. And interpretion is very important for works related to physics or [fairness](https://ieeexplore.ieee.org/abstract/document/9113719/).

## Reference

[Srivastava, N., Hinton, G., Krizhevsky, A., Sutskever, I., & Salakhutdinov, R. (2014). Dropout: a simple way to prevent neural networks from overfitting. *The journal of machine learning research*, *15*(1), 1929-1958.](https://www.jmlr.org/papers/volume15/srivastava14a/srivastava14a.pdf?utm_content=buffer79b43&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer,)

[Zeiler, M. D., & Fergus, R. (2014, September). Visualizing and understanding convolutional networks. In *European conference on computer vision* (pp. 818-833). Springer, Cham.](https://link.springer.com/chapter/10.1007/978-3-319-10590-1_53)

[Du, M., Yang, F., Zou, N., & Hu, X. (2020). Fairness in deep learning: A computational perspective. *IEEE Intelligent Systems*, *36*(4), 25-34.](https://ieeexplore.ieee.org/abstract/document/9113719/)
