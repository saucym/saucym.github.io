#`Flutter`组件解析之`Container`
`Flutter`使用声明式来写`UI`，跟`SwiftUI`类似，非常先进。但是与`HTML`或以前的`iOS`布局方式差异相当大，这里主要通过介绍万金油组件[`Container`](https://book.flutterchina.club/chapter5/container.html)来熟悉这种写法。

## 一、源码解析
`Container`实现了很多功能，很多时候有多种需要的时候一个`Container`就可以搞定。
我们来看看`Container`的创建源码，每个参数其实都是嵌套了一层小组件，这里把相应的小组件写在后面注释里面了。
```Dart
Container({
  Key key,
  this.alignment,               // Align
  this.padding,                 // Padding
  this.color,                   // ColoredBox
  this.decoration,              // DecoratedBox
  this.foregroundDecoration,    // DecoratedBox
  double width,                 // ConstrainedBox
  double height,                // ConstrainedBox
  BoxConstraints constraints,   // ConstrainedBox
  this.margin,                  // Padding
  this.transform,               // Transform
  this.child,
  this.clipBehavior = Clip.none, // ClipPath
})
```
可以点进去看`build`方法，这里列出部分代码
```Dart
Widget current = child;
...
final EdgeInsetsGeometry effectivePadding = _paddingIncludingDecoration;
if (effectivePadding != null)
  current = Padding(padding: effectivePadding, child: current);
  
if (color != null)
  current = ColoredBox(color: color, child: current);
  
if (constraints != null)
  current = ConstrainedBox(constraints: constraints, child: current);

if (margin != null)
  current = Padding(padding: margin, child: current);
```
比如
```Dart
Padding(padding: padding, child: ColoredBox(color: color, child: child))
```
用`Container`表示就是下面这样的
```Dart
Container(color: color, padding: padding, child: child)
```
不止是`Container`，还有很多其他组件其实也都是做了一些嵌套而已，这也是`声明式UI`的一大特色。在`Flutter`中，`Container`组件也正是组合优先于继承的实例。

## 二、属性介绍
### 1、`alignment`
对齐方向

需要注意的是`Container`默认大小是随子组件的，如果设置了这个`alignment`参数后`Container`会变成大小会跟随父组件。

### 2、`width`、`height`、`constraints`
这三个属性会合成一个约束通过ConstrainedBox嵌套。
它们同时出现的时候`width`、`height`优先。

### 3、`padding`和`margin`
这两个比较容易用混，我们通过下面一个例子来理解
![padding_margin](assets/padding_margin.jpg)
源码如下
```Dart
Center(
  child: Container(
    color: Colors.black,
    child: Container(
      color: Colors.red,
      margin: EdgeInsets.all(20),
      padding: EdgeInsets.all(10),
      child: Container(color: Colors.yellow, height: 10, width: 10),
    ),
  ),
)
```
可以看到红色部分就是`padding`，在`child`的外面扩充。
黑色部分就是`margin`产生的，可以理解为是我虽然只有红色这么大，但是上层你必须给我黑色这么大的空间。

其实`build`里面的源码嵌套顺序也可以看出来，先设置`padding`，然后再设置背景颜色和装饰这些，最后再设置`margin`，相当于最后再在外面套了一个空框。

### 4、`color`和 `decoration`
都是设置背景，`color`和 `decoration`是互斥的，两个只能设置一个， `decoration`功能更强大，可以设置非常炫酷的效果。

### 5、`foregroundDecoration`
前台装饰，先绘制背景和`child`，最后再绘制 `foregroundDecoration`，会挡住`child`。

### 6、`transform`
矩阵变换，可以是移动、缩放、绕x或y或z轴旋转、或者多个复合的变换。

### 7、`clipBehavior`枚举
`none`：无模式`hardEdge`：裁剪速度稍快，但容易失真，有锯齿。
`antiAlias`：裁剪边缘抗锯齿，使得裁剪更平滑，这种模式裁剪速度比`antiAliasWithSaveLayer`快，但是比`hardEdge`慢，该模式常用于圆形和弧形之类的形状裁剪。
`antiAliasWithSaveLayer`：裁剪后具有抗锯齿特性并分配屏幕缓冲区，所有后续操作在缓冲区进行，然后再进行裁剪和合成。

默认是`none`，这个属性一般情况下用不到。

结束语：`Container`是非常常用且实用的小组件，很有必要深入理解它。
