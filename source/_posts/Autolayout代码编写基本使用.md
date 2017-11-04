---
title: Autolayout代码编写基本使用
date: 2017-10-15 15:47:37
tags: iOS, Autolayout
categories: iOS
---

## 第一种
![效果图1](/img/iOS/autolayout/autolayout_figure_1.png)

代码如下：
```
    UIView *redView = [[UIView alloc] init];
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    redView.backgroundColor = UIColor.redColor;
    [self.view addSubview:redView];
```
```     
    //设置高度
    [redView addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:40.0]];
    
    //设置左边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeLeading relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeLeading multiplier:1.0 constant:20.0]];
    
    //设定顶边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTop multiplier:1.0 constant:20.0]];
    
    //设置右边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTrailing relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTrailing multiplier:1.0 constant:-20.0]];
```
换做使用VFL的话，代码如下：
```
    NSDictionary *views = NSDictionaryOfVariableBindings(redView);
    CGFloat height = 40;
    CGFloat margin = 20;
    NSDictionary *metrics = @{ @"margin":@(margin), @"height":@(height) };
    
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|-(margin)-[redView]-(margin)-|" options:0 metrics:metrics views:views]];
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-(margin)-[redView(==height)]" options:0 metrics:metrics views:views]];
```



<br/>

## 第二种
![效果图2](/img/iOS/autolayout/autolayout_figure_2.png)
```
   UIView *redView = [[UIView alloc] init];
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    redView.backgroundColor = UIColor.redColor;
    [self.view addSubview:redView];
    
    UIView *blueView = [[UIView alloc] init];
    blueView.translatesAutoresizingMaskIntoConstraints = NO;
    blueView.backgroundColor = UIColor.blueColor;
    [self.view addSubview:blueView];
```

``` 
    //创建redView的约束
    //设置高度
    [redView addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:40.0]];
    
    //设置左边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeLeading relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeLeading multiplier:1.0 constant:20.0]];
    
    //设定顶边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTop multiplier:1.0 constant:20.0]];
    
    //设定右边边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTrailing relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTrailing multiplier:1.0 constant:-20.0]];
    
    //创建blueView的约束
    //设置redView和blueView高度相等
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeHeight multiplier:1.0 constant:0.0]];
    
    //设置redView和blueView的间距为20
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:redView attribute:NSLayoutAttributeBottom multiplier:1.0 constant:20.0]];
    
    //让blueView和redView的右对齐
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeRight multiplier:1.0 constant:0.0]];
    
    //blueView的宽度是redView宽度的一半
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:redView attribute:NSLayoutAttributeWidth multiplier:0.5 constant:0.0]];
```


<br/>

## 第三种
![效果图3](/img/iOS/autolayout/autolayout_figure_3.png)
```
    UIView *redView = [[UIView alloc] init];
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    redView.backgroundColor = UIColor.redColor;
    [self.view addSubview:redView];
    
    UIView *blueView = [[UIView alloc] init];
    blueView.translatesAutoresizingMaskIntoConstraints = NO;
    blueView.backgroundColor = UIColor.blueColor;
    [self.view addSubview:blueView];
```

``` 
    //创建redView的约束
    //设置高度
    [redView addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:40.0]];
    
    //设置左边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeLeading relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeLeading multiplier:1.0 constant:20.0]];
    
    //设定顶边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTop multiplier:1.0 constant:20.0]];
    
    //创建blueView的约束
    //设置redView和blueView的高度相等
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeHeight multiplier:1.0 constant:0.0]];
    
    //设置redView和blueView的宽度相等
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeWidth multiplier:1.0 constant:0.0]];
    
    //设置blueView的顶部与redView对齐
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:redView attribute:NSLayoutAttributeTop multiplier:1.0 constant:0.0]];
    
    //设置右边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeTrailing relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTrailing multiplier:1.0 constant:-20.0]];
    
    //设置左边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTrailing relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeLeading multiplier:1.0 constant:-20.0]];
```
换做使用VFL的话，代码如下：
```
    NSDictionary *views = NSDictionaryOfVariableBindings(redView, blueView);
    CGFloat height = 40;
    CGFloat margin = 20;
    NSDictionary *metrics = @{ @"margin":@(margin), @"height":@(height) };
    
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|-(margin)-[redView]-(margin)-[blueView(==redView)]-(margin)-|" options:0 metrics:metrics views:views]];
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-(margin)-[redView(==height)]" options:0 metrics:metrics views:views]];
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-(margin)-[blueView(==redView)]" options:0 metrics:metrics views:views]];
```


<br/>

## 第四种
![效果图4](/img/iOS/autolayout/autolayout_figure_4.png)
```
    UIView *redView = [[UIView alloc] init];
    redView.translatesAutoresizingMaskIntoConstraints = NO;
    redView.backgroundColor = UIColor.redColor;
    [self.view addSubview:redView];
    
    UIView *blueView = [[UIView alloc] init];
    blueView.translatesAutoresizingMaskIntoConstraints = NO;
    blueView.backgroundColor = UIColor.blueColor;
    [self.view addSubview:blueView];
    
    UIView *greenView = [[UIView alloc] init];
    greenView.translatesAutoresizingMaskIntoConstraints = NO;
    greenView.backgroundColor = UIColor.greenColor;
    [self.view addSubview:greenView];
    
```

```
    //创建redView的约束
    //设置高度
    [redView addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:40.0]];
    
    //设置左边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeLeading relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeLeading multiplier:1.0 constant:20.0]];
    
    //设定顶边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTop multiplier:1.0 constant:20.0]];
    
    
    //创建blueView的约束
    //设置redView和blueView的高度相等
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeHeight multiplier:1.0 constant:0.0]];
    
    //设置redView和blueView的宽度相等
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeWidth multiplier:1.0 constant:0.0]];
    
    //设置blueView的顶部与redView对齐
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:redView attribute:NSLayoutAttributeTop multiplier:1.0 constant:0.0]];
    
    //设置左边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeTrailing relatedBy:NSLayoutRelationEqual toItem:blueView attribute:NSLayoutAttributeLeading multiplier:1.0 constant:-20.0]];
    

    //创建greenView的约束
    //设置redView和greenView的高度相等
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:greenView attribute:NSLayoutAttributeHeight multiplier:1.0 constant:0.0]];
    
    //设置redView和greenView的宽度相等
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:redView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:greenView attribute:NSLayoutAttributeWidth multiplier:1.0 constant:0.0]];
    
    //设置greenView的顶部与redView对齐
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:greenView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:redView attribute:NSLayoutAttributeTop multiplier:1.0 constant:0.0]];
    
    //设置右边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:greenView attribute:NSLayoutAttributeTrailing relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTrailing multiplier:1.0 constant:-20.0]];
    
    //设置左边距
    [self.view addConstraint:[NSLayoutConstraint constraintWithItem:blueView attribute:NSLayoutAttributeTrailing relatedBy:NSLayoutRelationEqual toItem:greenView attribute:NSLayoutAttributeLeading multiplier:1.0 constant:-20.0]];
```

换做使用VFL的话，代码如下：
``` 
    NSDictionary *views = NSDictionaryOfVariableBindings(redView, blueView, greenView);
    CGFloat height = 40;
    CGFloat margin = 20;
    NSDictionary *metrics = @{ @"margin":@(margin), @"height":@(height) };
    
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|-(margin)-[redView]-(margin)-[blueView(==redView)]-(margin)-[greenView(==redView)]-(margin)-|" options:0 metrics:metrics views:views]];
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-(margin)-[redView(==height)]" options:0 metrics:metrics views:views]];
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-(margin)-[blueView(==redView)]" options:0 metrics:metrics views:views]];
    [self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"V:|-(margin)-[greenView(==redView)]" options:0 metrics:metrics views:views]];
```

