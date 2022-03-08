---
title: learning rate schedule
author: daydreamatnight
toc: true
declare: true
date: 2022-03-08 09:32:32
tags:
	- deep learning
	- deep learning tricks
---

>Recently, I joined a [Kaggle image classification competition](https://www.kaggle.com/c/classify-leaves/), I used the pretrained ResNet50 plus other tricks and here is to record some of them I've learned for now.

> Learning rate schedule is one commonly used trick to control the process of training. Different kinds of learning tricks are presented every day. In this article, I have put together classical methods theories and apply them in this little competition.

<!-- more -->

### Introduction

Learning rate is one critical parameter in all iterative algorithms which can be used in PDE and ODE solving, optimization and eigenvalue calculation. In deep learning area, learning rate is more than critical because of the notorious difficulty on training.

Strictly, there are two ways of adjusting the learning rate: 

- learning rate scheduling, adjust the global learning rate during iteration
- adaptive learning rate, adjust the learning rate for each parameter base on their gradients updates(moments), also called adaptive gradient or gradient descent optimization.

But in this article, **only learning rate scheduling is mainly discussed**. And afterwards, learning rate refers to the global learning rate.

### Methods of learning rate scheduling

Apart from the constant learning rate there are several way to schedule the learning rate:

- change with epoch number
  --  learning rate decay: linear, step.

  -- learning rate decay with restart: cosine annealing 

  -- warmup
  
- change on some validation measurements: plateau

  #### learning rate decay

  Because of the the existence of stochastic noise, the overall gradient descent process is not straightforward. With constant learning rate, as shown in the gradient contour map below, the minima can not be reached with constant step (blue) because of the relatively large step at the bottom. And a lower minima can be reached if learning rate descend with the gradient i.e. the epoch(green).
  
  <img src="SGD%20with%20learning%20rate%20decay.png" alt="SGD with learning rate decay" style="zoom:50%;" />
  
  How to decay is a personal choice. It can be continuous or step, linear or polynomial, exponential, or trigonometric.
  
  #### learning rate decay with restart
  
  Because of the nonconvexity, it is a common sense that reaching a global minima is impossible. With standard leaning rate, a unstable local minima is more possible to trap the descending process as shown below. But a "learning rate restart" or "cyclic learning rate" would allow the process to  “jump” from one local minimum to another.
  
  <img src="2d%20cyclic%20learning%20rate%20schedule.png" alt="2d cyclic learning rate schedule" style="zoom:80%;" />
  
  <img src="cyclic%20learning%20rate%20schedule.png" alt="cyclic learning rate schedule" style="zoom:80%;" />
  
  Still there are several choices, but Cosine Annealing and Cosine Annealing Warm Restarts are more common.
  
  #### learning rate warmup
  
   Learning rate warmup is first use in the famous [Resnet](https://openaccess.thecvf.com/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf) paper
  
  > In this case, we find that the initial learning rate of 0.1 is slightly too large to start converging5 . So we use 0.01 to warm up the training until the training error is below 80% (about 400 iterations), and then go back to 0.1 and continue training.  
  
  And later [Goyal and He's work](https://arxiv.org/pdf/1706.02677.pdf) makes a major influence, where constant and gradual methods of warmup are discussed. And gradual warmup is proved to be effective on large minibatch size.
  
  > As we discussed, for large minibatches (e.g., 8k) the linear scaling rule breaks down when the network is changing rapidly, which commonly occurs in early stages of training. We find that this issue can be alleviated by a properly designed warmup [16], namely, a strategy of using less aggressive learning rates at the start of training. 
  
  <img src="warmup%20on%20large%20batches.png" alt="warmup on large batches" style="zoom:80%;" />
  
  In practice, warmup are always combined with other learning rate methods afterwards. And linear warmup is a default method.
  
  #### Reducing the learning rate on plateau
  
  Apart from methods scheduling the learning rate with epoch, a dynamic learning rate decay method is also a choice. It refers to the process of decaying the learning rate only when the optimizer can't lift the accuracy, or decrease the loss in serval epochs. 
  
### Apply lr scheduling in PyTorch

> `torch.optim.lr_scheduler` provides several methods to adjust the learning rate based on the number of epochs. 

For example, 

```python
def train_ch6(net, train_iter, test_iter, num_epochs, lr, device):
    print('training on', device)
    net.to(device)
    optimizer = torch.optim.Adam(net.parameters(), lr=lr)
    scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, num_epochs*len(train_iter)/10, eta_min=1e-9)
    loss = LSR(0.1) 
    for epoch in range(num_epochs):
        net.train()
        for i, (X, y) in enumerate(train_iter):
            X, y = X.to(device), y.to(device)
            optimizer.zero_grad()
            y_hat = net(X)
            l = loss(y_hat, y)
            l.backward()
            optimizer.step()
            scheduler.step()
```

Apart from well defined `lr_scheduler` ,  `torch.optim.lr_scheduler.LambdaLR` allow us to apply self define scheduler such as:

```python
print('training on', device)
net.to(device)
optimizer = torch.optim.Adam(net.parameters(), lr=lr)

t=10*len(train_iter)#warmup
T=num_epochs*len(train_iter)
lambda1 = lambda epoch: (0.9*epoch / t+0.1) if epoch < t else  0.1  if 0.5 * (1+math.cos(math.pi*(epoch - t)/(T-t)))<0.1 else 0.5 * (1+math.cos(math.pi*(epoch - t)/(T-t)))

scheduler = torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda=lambda1)

# plot learningrate_decay
lr_plot = []
for _i in range(num_epochs):
    for _j in range(len(train_iter)):
        optimizer.step()
        lr_plot.append(optimizer.param_groups[0]["lr"])
        scheduler.step()
plt.plot(lr_plot)
```
### Should we do scheduling with adaptive learning rate method?

From [Should we do learning rate decay for adam optimizer](https://stackoverflow.com/questions/39517431/should-we-do-learning-rate-decay-for-adam-optimizer), I found it as a arguable question.

>It depends. ADAM updates any parameter with an individual learning rate. This means that every parameter in the network has a specific learning rate associated. 
>
>But* the single learning rate for each parameter is computed using lambda (the initial learning rate) as an upper limit. This means that every single learning rate can vary from 0 (no update) to lambda (maximum update).
>
>It's true, that the learning rates adapt themselves during training steps, but if you want to be sure that every update step doesn't exceed lambda you can than lower lambda using exponential decay or whatever. It can help to reduce loss during the latest step of training, when the computed loss with the previously associated lambda parameter has stopped to decrease.

>  Yes, absolutely. From my own experience, it's very useful to Adam with learning rate decay. Without decay, you have to set a very small learning rate so the loss won't begin to diverge after decrease to a point.

In the article [Decoupled weight decay regularization](https://arxiv.org/abs/1711.05101)(adamW), it is encouraged.

> Adam can substantially benefit from a scheduled learning rate multiplier. The fact that Adam is an adaptive gradient algorithm and as such adapts the learning rate for each parameter does not rule out the possibility to substantially improve its performance by using a global learning rate multiplier, scheduled, e.g., by cosine annealing.  

All in all, theoretically, the adaptive learning rate methods such as adam adjust the learning rate for each parameters under a upper limit as the global learning rate, which can be adjusted by scheduling. In practice, it really depends. And if adamW is in use, just do what the article above encouraged.

### Experiment: adam vs adam + cosine annel



### Reference

[Adam: A Method for Stochastic Optimization](https://arxiv.org/pdf/1412.6980.pdf)

[Adaptive Learning Rate Method - TUM Wiki-System](https://wiki.tum.de/display/lfdv/Adaptive+Learning+Rate+Method#:~:text=Adaptive learning rate methods are,the parameters of the network.) 

[Learning Rate Schedules and Adaptive Learning Rate Methods](https://towardsdatascience.com/learning-rate-schedules-and-adaptive-learning-rate-methods-for-deep-learning-2c8f433990d1) 

[Learning Rate Decay and methods in Deep Learning](https://medium.com/analytics-vidhya/learning-rate-decay-and-methods-in-deep-learning-2cee564f910b#:~:text=Learning rate decay is a,help both optimization and generalization.) 

[A Newbie’s Guide to Stochastic Gradient Descent With Restarts](https://towardsdatascience.com/https-medium-com-reina-wang-tw-stochastic-gradient-descent-with-restarts-5f511975163)

[Deep Residual Learning for Image Recognition](https://openaccess.thecvf.com/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf) 

[Acurate, Large Minibatch SGD: Training ImageNet in 1 Hour](https://arxiv.org/pdf/1706.02677.pdf)

[torch.optim — PyTorch 1.10 documentation](https://pytorch.org/docs/stable/optim.html) 

[Should we do learning rate decay for adam optimizer](https://stackoverflow.com/questions/39517431/should-we-do-learning-rate-decay-for-adam-optimizer)

[Decoupled weight decay regularization](https://arxiv.org/abs/1711.05101)

[Guide to Pytorch Learning Rate Scheduling](https://www.kaggle.com/isbhargav/guide-to-pytorch-learning-rate-scheduling)