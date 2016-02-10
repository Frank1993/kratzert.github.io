---
layout: "post"
title: "Understanding the backwardpass through Batchnorm-Layer"
date: "2016-02-09 14:54"
excerpt: "An explenation of gradient flow through Batchnorm-Layer following the circuit represantation learned in Standfords class CS231n "
---
At the moment there is a wonderful course running at Standford University, called [Cs231n - Convolutional Neural Networks for Visual Recognition](http://cs231n.stanford.edu/), held by Andrej Karpathy, Justin Johnson and Fei-Fei Li. Fortunately all the [course material](http://cs231n.stanford.edu/syllabus.html) is provided for free and all the lectures are recorded and uploaded on [Youtube](https://www.youtube.com/playlist?list=PLkt2uSq6rBVctENoVBg1TpCC7OQi31AlC). This class gives and wonderful intro to machine learning/deep learning coming along with programming assignments.

## Batch Normalization

One Topic, which kept me quite buisy for some time was the implemantation of [Batch Normalization](http://arxiv.org/abs/1502.03167), especially the backward pass. Batch Normalization is a technique to provide any layer in a Neural Network inputs that are zero mean/unit variance - and this is basically what they like! But BatchNorm consists of one more step with makes this algorithm really powerful. Here is the the BatchNorm Algorithm:

<div class="fig figcenter fighighlight">
  <img src="/images/bn_backpass/bn_algorithm.PNG" width=400>
  <div class="figcaption"> Algorithm of Batch Normalization copied from the Paper by Ioffe and Szegedy mentioned above.<br><br>
  </div>
</div>


Look at the last line of the Algorithm. After normalizing the input `x` the result is squashed through a linear function with parameters `gamma` and `beta`. These are learnable parameters of the BatchNorm Layer and make it basically possible to say "Hey!! I dont want zero mean/unit variance input, give me back the raw input - it's better for me." If `gamma = sqrt(var(x))` and `beta = mean(x)`, the original activation is restored. This is, what makes BatchNorm really powerful in my opinion. We initialize the BatchNorm Parameters to transform the input to zero mean/unit variance distributions but during training they can learn that any other distribution might be better.
Anyway, I don't want to spend to much time on explaining Batch Normalization. If you want to learn more about it, the [paper](http://arxiv.org/abs/1502.03167) is very well written and [here](https://youtu.be/gYpoJMlgyXA?list=PLkt2uSq6rBVctENoVBg1TpCC7OQi31AlC&t=3078) Andrej is explaining BatchNorm in class.

Btw: it's called "Batch" Normalization because we perform this transformation and calculate the statistics only for a subpart (a batch) of the entire trainingsset.

## Backpropagation

In this blog post I don't want to give a lecture in Backpropagation and Stochastic Gradient Descent (SGD). For now I will assume that whoever will read this post, has some basic understanding of these principles. For the rest, let me quote Wiki:

>Backpropagation, an abbreviation for "backward propagation of errors", is a common method of training artificial neural networks used in conjunction with an optimization method such as gradient descent. The method calculates the gradient of a loss function with respect to all the weights in the network. The gradient is fed to the optimization method which in turn uses it to update the weights, in an attempt to minimize the loss function.

Uff, sounds tough, eh? I will maybe write another post about this topic but for now I want to focus on the concrete example of the backwardpass through the BatchNorm-Layer.

## Computational Graph of Batch Normalization Layer

I think one of the things I learned from the cs231n class that helped me most understanding backpropagation was the explenation through computational graphs. These Graphs are a good way to visualize the computational flow of fairly complex functions by small, piecewise differentiable subfunctions. For the Batch Normalization Layer it would look something like this:

<div class="fig figcenter fighighlight">
  <img src="/images/bn_backpass/BNcircuit.PNG">
  <div class="figcaption"> Computational graph of the BatchNorm-Layer. From left to right, following the black arrows flows the forward pass. The inputs are a matrix X and gamma and beta as vectors. From right to left, following the red arrows flows the backward pass which distributes the gradient from above layer to gamma and beta and all the way back to the input.<br><br>
  </div>
</div>

I think for all, who followed the course or who know the technique the forwardpass (black arrows) is easy and straightforward to read. From input `x` we calculate the mean of every dimension in the feature space and then substract this vector of mean values from every training example. With this done, following the lower branch, we calculate the per-dimension variance and with that the entire denominator of the normalization equation. Next we invert it and multiply it with differenz of inputs and means and we have `x_normalized`. The last two blobs on the left perform the squashing by multiplying with the input `gamma` and finally adding `beta`. Et voilà, we have our Batch-Normalized output.

A vanilla implemantation of the forwardpass might look like this:

```python
def batchnorm_forward(x, gamma, beta, eps):

  N, D = x.shape

  #step1: calciulate mean
  mu = 1./N * np.sum(x, axis = 0)

  #step2: substract mean vector of every trainings example
  xmu = x - mu

  #step3 - lower branch, calculation denominator
  sq = xmu ** 2

  #step4: calculate variance
  var = 1./N * np.sum(sq, axis = 0)

  #step5: add eps for numerical stability, then sqrt
  sqrtvar = np.sqrt(var + eps)

  #step6: invert sqrtwar
  ivar = 1./sqrtvar

  #step7: execute normalization
  xhat = xmu * ivar

  #step8: Nor the two transformation steps
  gammax = gamma * xhat

  #step9
  out = gammax + beta

  #store intermediate
  cache = (xhat,gamma,xmu,ivar,sqrtvar,var,eps)

  return out, cache
```

Note that for the excercise of the cs231n class we had to do a little more (calculate running mean and variance as well as implement different forward pass for trainings mode and test mode) but for the explenation of the backwardpass this piece of code will work.