---
title: Intro and Pytorch Implementation of Label Smoothing Regularization (LSR)
author: daydreamatnight
toc: true
declare: true
date: 2022-03-04 13:59:58
tags:
	- deep learning
	- deep learning tricks

---

> Recently, I joined a [Kaggle image classification competition](https://www.kaggle.com/c/classify-leaves/), I used the pretrained ResNet50 plus other tricks and here is to record some of them I've learned for now. 

> Soft label is a commonly used trick to prevent overfitting. It can always gain some extra points on the image classification tasks. In this article, I have put together useful information from theory to implementation of it.

<!-- more -->

### Introduction: from hard label to soft label

In deep learning, the neural network is basically a super powerful non-linear regression machine aimed to fit a function between the input and the label. And the result is always called label.

Hard label, in another word:  the one-hot vector, is the most commonly type of label that is used. For example, in this [Kaggle image classification competition](https://www.kaggle.com/c/classify-leaves/), to digitalize the different name of the leaves, it is intuitive to encode the leaves categories as: 0, 1, 2, 3. And the factorized target labels would be somehow like [1,3,0...] where each element stands for the categories of the data. With the resulting category dictionary, it can be easily decoded after the training.

<img src="leave%20class%20code.png" alt="leave class code" style="zoom:22%;" />

Actually, there is a slightly difference in the binary world. What usually do is, the previously factorized label will be extended to be a 2-dimensional "on-hot" matrix where the elements stands for the probability of each class. And the network is aimed to train itself to make inference label nearest to the target label.

<img src="hard%20label.png" alt="hard label" style="zoom:22%;" />

Soft label is just slightly deteriorate the strong one-hot label into a weaker one.

<img src="soft%20label.png" alt="soft label" style="zoom:22%;" />

### Simple explanation: How loss function lost information?

In the cross entropy loss function, where `y_inference` and `y_grountruth` stands for inference and target label, n stands for the number of class.

<img src="Cross%20entropy%20loss%20function.png" alt="Cross entropy loss function" style="zoom:22%;" />

With the one-hot label, the components are 0 except for the true category. In a other word, the `y_inference` of the wrong category is not considered at all i.e. the information of the wrong category is lost. Which is against the real word classification. 

### Effectiveness: Visualization

In [When does label smoothing help?](https://arxiv.org/pdf/1906.02629.pdf)  Hinton shows the feature map difference between without and with LSR:

<img src="Label%20smoothing%20feature%20norm.png" alt="Label smoothing feature norm" style="zoom:80%;" />

>- When label smoothing is applied, the clusters are much tighter because label smoothing encourages that each example in the training set is to be equidistant from all other class’s templates.
>- With hard targets, the clusters for semantically similar classes (for example different breed of dogs in ImageNet), are isotropic whereas, with label smoothing, clusters lie in an arc as shown in the third row. If you mix two semantically similar classes with a third semantically different class, the clusters are still much better than the ones obtained with hard targets as shown in the fourth row.

### Experiment: apply in competition

Label smoothing can be easily applied in [Tensorflow](https://www.tensorflow.org/api_docs/python/tf/keras/losses/CategoricalCrossentropy), but there is no such thing in PyTorch.  So overwrite the Cross-entropy loss function with LSR (implemented in 2 ways): 

```python
class LSR(nn.Module):
    """NLL loss with label smoothing.
    """
    def __init__(self, smoothing=0.0):
        """Constructor for the LSR module.
        :param smoothing: label smoothing factor
        """
        super(LSR, self).__init__()
        self.confidence = 1.0 - smoothing
        self.smoothing = smoothing

    def forward(self, x, target):
        logprobs = torch.nn.functional.log_softmax(x, dim=-1)
        nll_loss = -logprobs.gather(dim=-1, index=target.unsqueeze(1))
        nll_loss = nll_loss.squeeze(1)
        smooth_loss = -logprobs.mean(dim=-1)
        loss = self.confidence * nll_loss + self.smoothing * smooth_loss
        return loss.mean()
    
loss = LSR(0.1)
```

```python
class LSR2(nn.Module):

    def __init__(self, e=0.01,reduction='mean'):
        super().__init__()

        self.log_softmax = nn.LogSoftmax(dim=1)
        self.e = e
        self.reduction = reduction

    def _one_hot(self, labels, classes, value=1):
        """
            Convert labels to one hot vectors

        Args:
            labels: torch tensor in format [label1, label2, label3, ...]
            classes: int, number of classes
            value: label value in one hot vector, default to 1

        Returns:
            return one hot format labels in shape [batchsize, classes]
        """
        #print("classes", classes)
        one_hot = torch.zeros(labels.size(0), classes)

        # labels and value_added  size must match
        labels = labels.view(labels.size(0), -1)
        value_added = torch.Tensor(labels.size(0), 1).fill_(value)

        value_added = value_added.to(labels.device)
        one_hot = one_hot.to(labels.device)

        one_hot.scatter_add_(1, labels, value_added)

        return one_hot

    def _smooth_label(self, target, length, smooth_factor):
        """convert targets to one-hot format, and smooth
        them.

        Args:
            target: target in form with [label1, label2, label_batchsize]
            length: length of one-hot format(number of classes)
            smooth_factor: smooth factor for label smooth

        Returns:
            smoothed labels in one hot format
        """
        #print("length", length)
        #print("smooth_fact", smooth_factor)
        one_hot = self._one_hot(target, length, value=1 - smooth_factor)
        one_hot += smooth_factor / length

        return one_hot.to(target.device)

    def forward(self, x, target):

        if x.size(0) != target.size(0):
            raise ValueError('Expected input batchsize ({}) to match target batch_size({})'
                             .format(x.size(0), target.size(0)))

        if x.dim() < 2:
            raise ValueError('Expected input tensor to have least 2 dimensions(got {})'
                             .format(x.size(0)))

        if x.dim() != 2:
            raise ValueError('Only 2 dimension tensor are implemented, (got {})'
                             .format(x.size()))
        #print("x: ", x)
        #print("target", target)

        smoothed_target = self._smooth_label(target, x.size(1), self.e)
        x = self.log_softmax(x)
        loss = torch.sum(- x * smoothed_target, dim=1)
        if self.reduction == 'none':
            return loss

        elif self.reduction == 'sum':
            return torch.sum(loss)

        elif self.reduction == 'mean':
            return torch.mean(loss)

        else:
            raise ValueError('unrecognized option, expect reduction to be one of none, mean, sum')
            
loss = LSR2(0.1)
```

Pretrained ResNet50 is in use

```shell
lr, num_epochs, batch_size = 0.01, 10, 256
```

<img src="accuracy%20curve%20compare%20label%20smoothing%20with%20hard%20label.png" alt="accuracy curve compare label smoothing with hard label" style="zoom:67%;" />

It can bee seen that the under same `random seed`, `batch_size`, `lr`, and `num_epochs`, the overall accuracy has a fascinating rise of 0.5. 

Then apply the LSR and run 50 epochs, with learning rate 0.005 and batch size 256, the result turns to be:

<img src="accuracy%20curve%20applying%20label%20smoothing.png" alt="accuracy curve applying label smoothing" style="zoom:67%;" />

It is a exciting improvement, but more tricks still in need.

### Conclusion

3 disadvantaged of the hard label:

- the relationship between the true label and the others is neglected, tend to be overfitting
- the model is tend to be over confident i.e. less generalizable
- more sensitive to label with noise, wrong labeled for example.

Several good things about label smoothing:

- data augmentation by add more information, compensates for the lack of supervisory signals 
- Improves generalizability
- Improves noise robust
- lower the feature norm
- Improves model calibration 

Bad things about label smoothing:

-  label smoothing can't give real relationship between labels. It simply adds random noise, under fitting might happen under certain scenarios.
- If distill in use, the teach network preforms worse when apply label smoothing, more explanation in  [When does label smoothing help?](https://arxiv.org/pdf/1906.02629.pdf) 

### Reference

 [标签平滑 - Label Smoothing概述 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1815786) 

 [大道至简：算法工程师炼丹Trick手册 (qq.com)](https://mp.weixin.qq.com/s?__biz=Mzg4MzU1NjQ2Mw==&mid=2247495228&idx=1&sn=ec685adcf8a274e8235c177718868a34&scene=21#wechat_redirect) 

[深度学习trick--labelsmooth](https://cloud.tencent.com/developer/article/1684298?from=article.detail.1815786)

 [Label Smoothing 标签平滑 (Label smooth regularization, LSR)_hxxjxw的博客-CSDN博客](https://blog.csdn.net/hxxjxw/article/details/115298103) 

 [When Does Label Smoothing Help?](https://medium.com/@nainaakash012/when-does-label-smoothing-help-89654ec75326)

[ suvojit-0x55aa](https://gist.github.com/suvojit-0x55aa)/[label_smoothing.py](https://gist.github.com/suvojit-0x55aa/0afb3eefbb26d33f54e1fb9f94d6b609)

