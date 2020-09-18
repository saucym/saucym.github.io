# 2. Wrap控件
```Dart
Wrap({
    Key key,
    this.direction = Axis.horizontal,
    this.alignment = WrapAlignment.start,
    this.spacing = 0.0,
    this.runAlignment = WrapAlignment.start,
    this.runSpacing = 0.0,
    this.crossAxisAlignment = WrapCrossAlignment.start,
    this.textDirection,
    this.verticalDirection = VerticalDirection.down,
    this.clipBehavior = Clip.hardEdge,
    List<Widget> children = const <Widget>[],
  })
```

# 3. 疑问
1. Flutter里面如何push一个类似alert的页面，让透明的地方的点击响应传给上一个页面
