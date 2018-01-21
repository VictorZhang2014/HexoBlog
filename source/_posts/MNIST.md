---
title: MNIST
date: 2018-01-20 16:23:16
tags: MNIST
categories: AI
---


## Softmax Regression(Softmax回归)
我们知道MNIST的每一张图片都表示一个数字，从0到9。我们希望得到给定图片代表每个数字的概率。
比如说，我们的模型可能推测一张包含9的图片代表数字9的概率是80%，但是判断它是8的概率是5%（因为8和9都有上半部分的小圆），
然后给与它代表其他数字的概率更小的值。

这是一个使用softmax回归（softmax regression）模型的经典案例。softmax模型可以用来给不同的对象分配概率。
即使在之后，我们训练更加精细的模型时，最后一步也需要用softmax来分配概率。




