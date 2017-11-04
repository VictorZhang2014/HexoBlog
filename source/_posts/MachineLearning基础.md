---
title: Machine Learning Basics
date: 2017-10-24 14:29:34
tags: Machine Learning
categories: AI
---

## 概述
机器学习指的是机器通过统计学算法，对大量的历史数据进行学习从而生成经验模型，利用经验模型指导业务。

## 机器学习算法分类：
- `有监督学习(Supervised)`：针对打过标签的数据去预测新出现的数据。回归和分类本质上是类似的，所以很多的算法既可以用作分类，也可以用作回归。
    - 回归(Regress)，如果预测的内容是数值类型，就称为回归；
        * 线性回归(Linear Regression)，
    - 分类(Classification)，如果预测的内容是类别或者是离散的，就称为分类
        * 支持向量机 SVM，
        * 随机森林
        * 神经网络
        * Gradient Boosting Tree
        * 决策树
        * 逻辑回归，虽然名字叫做回归，但它是分类算法
        * 朴素贝叶斯
        * KNN
- `无监督学习(Unsupervised)`：没有打过标签的数据就是无监督学习
    - 降维(Dimensionality Reduction) 
        * 就是把高维度的数据变成低维度，降维方法有PCA, LDA, SVD等
    - 聚类(Clustering)，就是把所有具有相同特质的数据归并在一起
        * K均值(K-Means)，一种聚类算法
        * 基于密度的聚类算法 DBSCAN(Density-Based Spatial Clustering of Applications with Noise)
- `半监督学习`：标签传播聚类
    - 有一部分数据打过标签，对这一部分数据进行学习

Caffe简单介绍
http://caffe.berkeleyvision.org








