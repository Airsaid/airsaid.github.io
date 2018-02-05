---
title: Android 约束布局（ConstraintLayout）详解
permalink: Android-ConstraintLayout-Detailed
date: 2018-02-05 11:32:48
categories:
	- Android
tags:
	- Android
	- ConstraintLayout
---


> 本篇 ConstraintLayout 讲解版本：1.0.2，1.1.x 版本开始新增功能在下篇进行讲解


<!-- more -->


## 前言

ConstraintLayout 是一个 ViewGroup，它的出现是为了解决复杂布局时，布局嵌套（布局内的布局）过多的问题（嵌套布局会增加绘制界面所需的时间）。它可以根据同级视图和父布局的约束条件为每个视图定义位置，类似于 RelativeLayout 所有视图都是根据兄弟视图和父级布局之间的关系来布局的，但是与 RelativeLayout 相比，它更加灵活，更易于使用。

在 Android 2.2 的版本中，为了给 ConstraintLayout 提供支持，Android 设计了新的布局编辑器。我们可以直接在布局编辑器当中拖动控件、添加约束，一气呵成。当然，在布局编辑器当中所做的操作，XML 布局当中就会自动生成对应的属性。

其中生成的一些属性，有可能并不是必须需要的。所以在生成后，你可能还需要手动检查、清理一下。或者你也可以直接在 XML 中编写属性，ConstraintLayout 的属性虽多，但是都是成组的，还是很容易掌握的。



## 约束概述

要在 ConstraintLayout 中定义 View 的位置，必须为该 View 添加至少一个水平和垂直约束（否则该 View 就会在左上角绘制）。该约束对象可以是另一个视图，或者父布局（也就是 ConstraintLayout），或者是不可见的 Guideline（后面会讲 Guideling）等。

如果有缺少的约束（或其他的一些优化问题），编辑器会在右上角显示警告提示（编译器不会错误）：

![警告提示（提示有 5 处可修复和优化的地方）](警告提示.png)


![正常情况](正常情况.png)



## 转换布局

如果要将现有布局转换为约束布局，只需按照下列步骤操作：

1. 在 Android Studio 中打开布局，然后单击编辑器窗口底部的 Design 选项卡。

2. 在 Component Tree 窗口中，右键单击布局，然后单击 Convert layout to ConstraintLayout。

![转换布局](转换代码.png)



## 开始

要在项目中使用 ConstraintLayout，我们需要在 build.gradle 中添加依赖（新版 AndroidStudio 会默认添加）：


``` java
compile 'com.android.support.constraint:constraint-layout:1.0.2'
```


然后，在工具栏中同步 Gradle 文件。



## 创建布局

接下来，我们创建一个布局，根布局就用 ConstraintLayout：



``` java
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">


</android.support.constraint.ConstraintLayout>
```



## 添加约束

添加约束十分简单，我们首先从 Design 界面的 Palette 操作栏拖动一个 Button 到蓝图中，添加完后，点击选中它，效果是这样的：

![蓝图效果](蓝图效果.png)

可以看到，上下左右都有一个小圆圈，这个圆圈就是用来添加约束的。

四个角的矩形，是用来扩大或缩小控件的。

![删除该控件上所有约束](删除该控件上所有约束.png)

![编辑基线，用于基线对齐（下面会说）](编辑基线.png)

现在我们来为这个 Button 添加约束：

![添加约束（约束对象为父控件）](添加约束.gif)

看这个效果，有没有一种「四马分尸」的感觉？当只有一条边拉这个 Button 时，Button 马上就被拉过去了，但是当四个边都开始拉这个 Button 时，Button 就在中间动弹不得了。

看完了动图，我们再来看看 XML 布局中的变化：



``` java
<Button
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button"
    app:layout_constraintTop_toTopOf="parent"
    android:layout_marginTop="8dp"
    android:layout_marginLeft="8dp"
    app:layout_constraintLeft_toLeftOf="parent"
    android:layout_marginRight="8dp"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    android:layout_marginBottom="8dp"/>
```


因为我们的目的是想让这个 Button 居中，但是它却默认给我们添加了 margin，这个显然是多余的。

如果不想每次添加约束后都在这条边的约束上添加 margin，我们可以在上面的工具栏中进行设置：

![设置控件默认的 margin](设置默认margin.png)

抛去 margin 相关的属性，那么约束父控件的相关属性就是如下这些了：



``` java
app:layout_constraintTop_toTopOf="parent"
```
``` java
app:layout_constraintLeft_toLeftOf="parent"
```
``` java
app:layout_constraintRight_toRightOf="parent"
```
``` java
app:layout_constraintBottom_toBottomOf="parent"
```


在上面的图中我们已经看到了，每个控件的上下左右都有一个圆圈，用于添加约束的。

那么这些属性的 constraintXXX 就是指定的当前控件的约束位置的，而 toXXX 就是目标约束位置。

这个位置可以是上面的例子一样是 parent（也就是父控件），也可以是另一个控件。

接下来我们再来看看对于其他控件的约束。

再拖一个 Buttton 进入蓝图中，然后给这个 Button 添加约束：

![添加约束（约束对象为另一个控件）](添加约束（约束对象为另一个控件）.gif)

XML 布局：

``` java

<Button    
	android:id="@+id/button2"    
	android:layout_width="wrap_content"    
	android:layout_height="wrap_content"    
	android:text="Button"    
	app:layout_constraintLeft_toLeftOf="@+id/button"    
	app:layout_constraintRight_toRightOf="@+id/button"    
	android:layout_marginTop="48dp"    
	app:layout_constraintTop_toBottomOf="@+id/button"/>
```

添加完约束了，如果要删除指定的单个约束的话，单击变红的圆圈即可：

![删除指定的单个约束](删除指定的单个约束.gif)

或者是上面提到了，删除指定控件的所有约束。

或者是工具栏中的，删除页面所有的约束：

![删除页面所有的约束](删除页面所有的约束.png)



## 基线约束

基线约束的作用就是将视图的文本基线与另一个视图的文本基线对齐：

![添加基线约束](添加基线约束.gif)

添加完成后，在 XML 布局中，B 控件就会多出这么一个属性：


``` java
app:layout_constraintBaseline_toBaselineOf="@+id/a"
```


该属性就是用于基线对齐的。



## Chains

Chains（链）是一种特定的约束，一个链包含了多个视图，它允许链中的视图共享空间，并控制可用空间在它们之间如何分配。该效果与 LinearLayout 的 Weiget 类似，但是链的作用远远超过它。

我们要是想创建一个链，那么首先就需要创建多个视图控件，然后再选择是创建「水平链」还是「垂直链」。

让我们从三个视图中创建一个「水平链」：

![创建「水平链」](创建「水平链」.gif)

「垂直链」：

![创建「垂直链」](创建「垂直链」.gif)

以「水平链」为例，我们来看看它们的样子：

![「水平链」](「水平链」.png)

可以发现，两侧控件的约束样子和前面约束的样子是一样的，我把它称之为「单约束」，也就是两侧控件只是单方面的约束了对方（这里是父控件）。

而中间的控件，两侧的约束则是链条的样子，这表示它约束了左右侧控件，而左右侧控件同样也约束了它：「互相约束」。

在 XML 布局中，是通过如下属性实现的：



``` java
<Button
    android:id="@+id/button1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toLeftOf="@+id/button2"
    tools:layout_editor_absoluteY="69dp"/>

<Button
    android:id="@+id/button2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button"
    app:layout_constraintLeft_toRightOf="@+id/button1"
    app:layout_constraintRight_toLeftOf="@+id/button3"
    tools:layout_editor_absoluteY="69dp"/>

<Button
    android:id="@+id/button3"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button"
    app:layout_constraintLeft_toRightOf="@+id/button2"
    app:layout_constraintRight_toRightOf="parent"
    tools:layout_editor_absoluteY="69dp"/>
```



我们刚刚创建好了一条链，此时我们可以设置 Cycle Chain Mode（链式模式）来告诉 Chains：“你应该怎样去填充剩余的空间”。

在「蓝图」中，设置的方式是点击链条的小图标：

![多出来了链条图标](多出来了链条图标.png)

在「布局」中，设置的属性是：

``` java
layout_constraintHorizontal_chainStyle
```
``` java
layout_constraintVertical_chainStyle
```

分别对应着「水平链」和「垂直链」。

有三种可选的参数：

- spread：将可用空间以均匀分布的方式将视图放置在链中（默认模式）：
![spread](spread.png)

- spread_inside：将链中最外面的视图对齐到外边缘，然后在可用空间内均匀的放置链中的其他视图：
![spread_inside](spread_inside.png)

- packed：将链中的视图紧紧的放在一起（可以提供边距让其分开），然后让其居中在可用空间内：
![packed](packed.png)

除了上面说的之外，Chains 还能对链中单独的视图控件设置 Weiget。因为目前并不支持在编辑器当中设置 Weiget，所以我们只能在布局中进行设置了，设置的属性是：

``` java
layout_constraintHorizontal_weight
```
``` java
layout_constraintVertical_weight
```

我们现在将第一个 Button 填充完剩余的空间，在该 Buttton 上设置如下属性：


``` java
android:layout_width="0dp"
```

``` java
app:layout_constraintHorizontal_weight="1"
```

![weight 效果](weight.png)

使用和 LinearLayout 的 Weiget 一致，很好理解。



## Properties

基本的添加、删除约束我们已经掌握了，那么现在再来将目光转向右侧的 Properties 区域。

![Properties](Properties.png)

该区域分为了上下两个部分，下部分就是选中控件的一些属性什么的，很好理解。

重点来看看上半部分，该部分被称为：Inspector 区域。

![Inspector](Inspector.png)

首先来看这些数字，很明显就是当前控件的 margin 值，我们可以直接在上面进行修改：

![修改控件的 margin](修改控件的margin.png)

还有就是左边和下边的两个轴，我把它称之为纵横轴。作用是用于确定控件位置的：

![修改纵横轴](修改纵横轴.gif)

根据上图可以看到，因为该控件同时添加了水平和垂直约束，所以两条轴都可以进行操作。

轴上面的数字是比例，对应的属性如下：



```
app:layout_constraintVertical_bias="0.5"
app:layout_constraintHorizontal_bias="0.5"
```


再来看看中心区域：

![Inspector 中心区域](Inspector中心区域.png)

四条边的原点就是添加的约束，点击的作用是删除该约束。

重要的是中心的 <<<，这个代表了当前控件的宽高计算方式。点击是可以切换的，一共有三种：

- ![](>>>.png)：这个代表的就是 wrap_content，不用过多解释了。

- ![](指定大小.png)：这个就是指定宽高的大小。

- ![](MatchConstraints.png)：这个代表的是：Match Constraints（有点类似于 match_parent），就给强翻成约束匹配吧... 这个是 ConstraintLayout 特有的，它会尽可能扩展以满足各方的约束（在考虑视图边界之后）。但是，可以使用以下属性和值修改该行为（只有在将视图宽度设置为与约束匹配时，这些属性才会生效）：
	- layout_constraintWidth_default：
		- spread：这是默认的行为，它会尽可能的扩展视图来满足约束。
		- wrap：与上面所说的 wrap_content 不同的是，虽然都是适应内容，但仍然允许视图比约束要求的视图更小。
		- layout_constraintWidth_min：指定视图的最小宽度（dp）。
​		- layout_constraintWidth_max：指定视图的最大宽度（dp）。


下面来几个例子，演示下约束匹配的效果，比如说，要让一个 Button 宽度填充满，仅需要将宽度设置为 0dp 即可：


``` java
android:layout_width="0dp"
```

如果是两个 Button：

``` java
<Button    
	android:id="@+id/button1"    
	android:layout_width="0dp"    
	android:layout_height="wrap_content"    
	android:text="A"    
	app:layout_constraintTop_toTopOf="parent"    
	app:layout_constraintLeft_toLeftOf="parent"    
	app:layout_constraintRight_toRightOf="parent"    
	app:layout_constraintBottom_toBottomOf="parent"/>

<Button    
	android:id="@+id/button2"    
	android:layout_width="0dp"    
	android:layout_height="0dp"    
	android:text="B"    
	app:layout_constraintTop_toBottomOf="@+id/button1"    
	app:layout_constraintBottom_toBottomOf="parent"    
	app:layout_constraintLeft_toLeftOf="parent"    
	app:layout_constraintRight_toRightOf="parent"/>

```

效果：

![约束匹配效果](约束匹配效果.png)



## 设置宽高比例

当宽高至少有一项设置为 0dp 时（也就是 Match Constraints），那么我们就可以为该视图设置宽高比例。设置的属性是：

``` java
layout_constraintDimensionRatio
```

如：

``` java
<Button    
	android:id="@+id/button"    
	android:layout_width="0dp"    
	android:layout_height="wrap_content"    
	android:text="button"    
	app:layout_constraintDimensionRatio="3:1"/>
```

![设置宽高比](设置宽高比.png)

参数是一个浮点值：“宽度：高度”，在该例子中，表示的宽高比是 3:1，也就是高度是宽度的三分之一。

当我们的宽高都设置为 0dp 时，如：

``` java
<Buttona
	android:id="@+id/button"    
	android:layout_width="0dp"    
	android:layout_height="0dp"    
	android:text="button"    
	app:layout_constraintLeft_toLeftOf="parent"    
	app:layout_constraintRight_toRightOf="parent"/>
```

此时，该视图是不会显示的，虽然我们加了 left 和 rigth 约束到父控件，让它知道了它的宽度就是父控件的高度，但是此时它还并不知道自己有多高。
所以此时我们在添加 ration 时，则可以指示哪一方应该受到约束，方法是在比率前面添加字母 W（用于限制宽度）或 H（用于限制高度），用逗号分隔：

``` java
app:layout_constraintDimensionRatio="h, 3:1"
```

这里设置的宽高比例是 3:1，那么高度就只有宽度的三分之一了：

![设置宽高比](设置宽高比2.png)



## Guidelines

Guidelines 是一条对实际显示界面不可见的线。用处是帮助我们的控件增加约束。

我们可以通过该图标来添加垂直或者水平的 Guidelines：

![添加 Guideline](添加Guideline.png)

比如我们想让控件在中心显示：

![Guideline 演示](Guideline.gif)

可以看到 Guidelines 可以切换百分比，或者是实际的距离。具体用哪种还是得根据实际情况来。



## 自动添加约束

有两种方式实现自动添加约束，分别是：

- Autoconnect：这是一个独立的功能，默认是关闭的。我们可以开启它，开启后，它将自动为新添加的视图创建两个或者更多的约束。其他的视图则不会自动添加，如：

![Autoconnect](Autoconnect.gif)

- Infer Constraints：为当前所有的视图自动添加约束：

![Infer Constraints](InferConstraints.gif)



## 参考

- [https://developer.android.com/training/constraint-layout/index.html?hl=zh-cn#add-constraintlayout-to-your-project](https://developer.android.com/training/constraint-layout/index.html?hl=zh-cn#add-constraintlayout-to-your-project)

- [https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html)

- [https://constraintlayout.com/](https://constraintlayout.com/)

- [http://blog.csdn.net/guolin_blog/article/details/53122387](http://blog.csdn.net/guolin_blog/article/details/53122387)