---
title: Swift颜色渐变 - 动画更新/过渡
date: 2017-11-08 22:16:49
tags: CAGradientLayer, CAAnimation
categories: iOS
---

## 1.简介
在这篇文章，我将会教你如何使用`CAGradientLayer`的API去创建一个非常炫酷的渐变效果，而且还可以是动画过渡效果。
比如现有的App，像：陌陌，Instagram，Snapchat等，都有一样的效果，非常好看，酷比~~

<br/>

## 2.渐变之 CAGradientLayer
`CAGradientLayer`是派生自`CALayer`，它允许我们绘制渐变效果；原理是它可以接收几种颜色作为输入源，然后混合这几种颜色到一起，并且渲染可持续的渐变颜色。

### 2.1 基本使用
下面的代码会在两个颜色之间创建一个斜角渐变，渐变的方向是通过`startPoint`和`endPoint`的属性来控制。
```
let gradient = CAGradientLayer()
gradient.frame = self.view.bounds
gradient.colors = [
    UIColor(red: 48/255, green: 62/255, blue: 103/255, alpha: 1).cgColor,
    UIColor(red: 244/255, green: 88/255, blue: 53/255, alpha: 1).cgColor
]
gradient.startPoint = CGPoint(x:0, y:0)
gradient.endPoint = CGPoint(x:1, y:1)
self.view.layer.addSublayer(gradient)
```
如图所示：
<img src="/img/iOS/animation/gradient01.png" alt="" width="320" height="560" />

<br/>

## 3.渐变动画
由于所有的`CALayer`的属性都是可动画形式的，所以我们可以创建一个`CABasicAnimation`实例，然后添加到`CAGraidentLayer`上，并且对`CABasicAnimation`的颜色属性设置相应的颜色，也就是从一种颜色到另一种颜色。
例如：
```
let gradientChangeAnimation = CABasicAnimation(keyPath: "colors")
gradientChangeAnimation.duration = 5.0
gradientChangeAnimation.toValue = [
    UIColor(red: 244/255, green: 88/255, blue: 53/255, alpha: 1).cgColor,
    UIColor(red: 196/255, green: 70/255, blue: 107/255, alpha: 1).cgColor
    ]
gradientChangeAnimation.fillMode = kCAFillModeForwards
gradientChangeAnimation.isRemovedOnCompletion = false
gradient.add(gradientChangeAnimation, forKey: "colorChange")
```
效果如下视频
<iframe src="https://player.vimeo.com/video/214767743" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>


### 3.1 一直持续的渐变动画
从上面的视频看出，该颜色只是切换了一次就结束了。。。其实我们还可以设置颜色不停的切换。
从这个方法中，我可以更好的扩展，创建一个循环式的动画，从一个渐变色无缝的过渡到另一个。实现这种代码也很简单，只需要我们的`ViewController`实现`CAAnimationDelegate`协议，和实现此方法`func animationDidStop(_ anim: CAAnimation, finished flag: Bool)`，代码如下

```
class ViewController: UIViewController, CAAnimationDelegate {
    
    let gradient = CAGradientLayer()
    var gradientSet = [[CGColor]]()
    var currentGradient: Int = 0
    var gradientChangeAnimation: CABasicAnimation?
    
    let gradientOne = UIColor(red: 48/255, green: 62/255, blue: 103/255, alpha: 1).cgColor
    let gradientTwo = UIColor(red: 244/255, green: 88/255, blue: 53/255, alpha: 1).cgColor
    let gradientThree = UIColor(red: 196/255, green: 70/255, blue: 107/255, alpha: 1).cgColor
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        gradientSet.append([gradientOne, gradientTwo])
        gradientSet.append([gradientTwo, gradientThree])
        gradientSet.append([gradientThree, gradientOne])
        
        
        gradient.frame = self.view.bounds
        gradient.colors = gradientSet[currentGradient]
        gradient.startPoint = CGPoint(x:0, y:0)
        gradient.endPoint = CGPoint(x:1, y:1)
        gradient.drawsAsynchronously = true
        self.view.layer.addSublayer(gradient)

        gradientChangeAnimation = CABasicAnimation(keyPath: "colors")
        gradientChangeAnimation!.duration = 5.0
        gradientChangeAnimation!.fillMode = kCAFillModeForwards
        gradientChangeAnimation!.isRemovedOnCompletion = false
        gradientChangeAnimation!.delegate = self
        
        animateGradient()
        
    }
    
    func animateGradient() {
        if currentGradient < gradientSet.count - 1 {
            currentGradient += 1
        } else {
            currentGradient = 0
        }
        
        gradientChangeAnimation!.toValue = gradientSet[currentGradient]
        gradient.add(gradientChangeAnimation!, forKey: "colorChange")
    }

    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        if flag {
            gradient.colors = gradientSet[currentGradient]
            animateGradient()
        }
    }
}
```
效果如下视频
<iframe src="https://player.vimeo.com/video/214767743" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

<br/>

## 总结
`CAGradientLayer`是一个非常强悍的API，经常被用来创建有关`CALayer`的炫酷动画以一种非常简单的方式。

如有疑问，欢迎在下方提问~


