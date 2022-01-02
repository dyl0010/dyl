---
title: Qt封装一个简单的LED(指示灯)控件
date: 2021-12-21 15:15:29
tags: [qt,widget]
---
### 效果~
![](https://pic.imgdb.cn/item/61c18ecc2ab3f51d9182065d.gif)

- 所以  
- 这个简单的LED类可以自定义大小~  
- 可以点亮或熄灭，也可以闪烁👁️~  
- 闪烁的时间间隔⌛可自定义，只需要在设置闪烁状态前调用类的setInterval(msec)函数~  
- 有几种预定义的颜色（红🔴、绿🟢、蓝🔵、黄🟡和橘色🟠）,也提供自定义颜色的函数~  
- 如果还满足不了需要，就得自己扩展🧑‍🔧了，类很小，这也很方便~  
- 工程文件中有这个例子代码，可以当作用法入门，其实只是引入头文件，定义类对象，然后当一个普通控件(如：QPushButton)来用~  
- 最简单的方法是把类文件QSimpleLed.h和QSimpleLed.cpp拷到自己工程里，然后在需要使用的文件里包- 含。类代码对Qt版本没有特别要求~  
- 差不多了，有了上面这几点提示，从下面下载文件try try💪💪💪，如果还有其它问题，欢迎在下面留个评论~  
- 另外下面第三部分是实现过程中的一些要点~  

### 资源文件
- 下载链接：链接：https://share.weiyun.com/5HK0503W 密码：rxdmyk  
 
![工程文件结构 ](https://pic.imgdb.cn/item/61c18ecc2ab3f51d91820660.png)

- `QtSimpleLed.h`和`QtSimpleLed.cpp`是封装的LED类  
- `mainwindow.h`和`mainwindow.cpp`里面有例子  
- 其它是Qt建工程时自己生成的。  

### 关于例子

```cpp
    QTimer::singleShot(200/7.0*1, nullptr, [&]() {
        stepLed01->setStates(QSimpleLed::BLINK);
    });
```

- 上面的代码是例子中设置LED闪烁的，用了lamda(C++11引入)，第一次接触可以不太懂，意思是：
- `200/7.0*1`秒后执行一下这个代码块`{ stepLed01->setStates(QSimpleLed::BLINK); }`
- 用传统写法：

```cpp
// 定义一个定时器
// 将定时器设置成一次触发(singleShot)类型
// 定义一个slot里面是 stepLed01->setStates(QSimpleLed::BLINK);
// 连接定时器的timeout信号和这个slot
// 调用start()启动定时器
```

- 所以这样步骤比较繁琐，第一次接触可以看看C++ lamda的语法，用起来非常容易，毕竟总是先有了第一次才会有第二次~🙂

### 实现过程关键点…

- 这地写实现过程中的一些想法。  
- 整体思路是继承`QAbstractButton`类然后实现其重绘(`paintEvent()`)虚函数，最后绘制我们想要的效果：
![](https://pic.imgdb.cn/item/61c18ecc2ab3f51d91820668.png)
- 下面的问题是怎样实现LED所表现出来的效果了:
- 这里主要用到了颜色渐变效果，这点我在之前的一个blog中说的足够多：[一文搞懂Qt颜色渐变](https://blog.csdn.net/weixin_37818081/article/details/118879134 )
- 这里用的是径向渐变，也非常容易想象出来，看下图理解:
![](https://pic.imgdb.cn/item/61c194a52ab3f51d91850adf.png)
- 所以在类中可以看到类似代码：

```cpp
    //
    // gradient - 1
    radialGent = QRadialGradient(QPointF(-500, -500)
                                 , 1500
                                 , QPointF(-500, -500));
    radialGent.setColorAt(0, QColor(224, 224, 224));
    radialGent.setColorAt(1, QColor(28, 28, 28));
	
    // ...

    //
    // gradient - 2
    radialGent = QRadialGradient(QPointF(500, 500)
                                 , 1500
                                 , QPointF(500, 500));
    radialGent.setColorAt(0, QColor(224, 224, 224));
    radialGent.setColorAt(1, QColor(28, 28, 28));
	
    // ...
    
    if (isChecked()) {
        //
        // gradient - on
        radialGent = QRadialGradient(QPointF(-500, -500)
                                     , 1500
                                     , QPointF(-500, -500));
        radialGent.setColorAt(0, smColorPalette[mColor].on0);
        radialGent.setColorAt(1, smColorPalette[mColor].on1);

    } else {
        //
        // gradient - off
        radialGent = QRadialGradient(QPointF(500, 500)
                                     , 1500
                                     , QPointF(500, 500));
        radialGent.setColorAt(0, smColorPalette[mColor].off0);
        radialGent.setColorAt(1, smColorPalette[mColor].off1);
    }
	...
```

- 最后一个问题，在重绘函数中使用了下面这种绘制方式：

```cpp
	...
    painter.setRenderHint(QPainter::Antialiasing);      
    painter.translate(width()/2, height()/2);           
    painter.scale(realSize/1000, realSize/1000);  // 这里对坐标系统进行缩放！！！
    painter.setBrush(QBrush(radialGent));
    painter.drawEllipse(QPointF(0, 0), 500, 500);
    ...
```

- 想象一下，如果想绘制一个内切于控件的圆形，我们可能会这样：

```cpp
 	painter.drawEllipse(QPointF(0, 0), realSize/2, realSize/2);
```

- 其中realSize是当前控件的实际大小(较小的一边)，我们用它的一半作为半径来绘制圆，形成内切。
- 但现在取而代之的是下面这两句：

```cpp
    painter.scale(realSize/1000, realSize/1000);  
    ...
    painter.drawEllipse(QPointF(0, 0), 500, 500);
```

- 这同样实现了绘制内切圆。  
- 我们实际上是换了一个思考角度，之前绘制时，我们总是把坐标系当做基准，我们所画的图形都来适应已存在的坐标系，现在我们用控件的实际大小来影响坐标系的构造，当我们绘制一个大小为`(500, 500)`的圆形时，在用图形大小(`realSize`)构造的这个坐标系内，它的实际大小是`realSize/1000 * 500`，也就是`1/2 * realSize`，这正是图形实际大小的一半。  
- 这样"反客为主“的方式也的确让我们有了绘制的主动权，否则我们所有与坐标有关的代码中都得引用realSize，容易出错也不易读。  
- 为什么除1000？其实这并不是重点，只是1000看起来更像一个单位，关键的一点是我们的确用realSize影响了坐标系的构造，也就是这句:  

```cpp
	painter.scale(realSize/1000, realSize/1000); 
```

- 在此坐标系下，我们绘制长度1000的直线，其实其长度只有realSize大小，此时我们关注更多的是比例，而不是具体的坐标值…  
### 参考
- [一个很棒的配色网站](https://www.schemecolor.com/)
- [https://github.com/colinbourassa/rovergauge/](https://github.com/colinbourassa/rovergauge/)
