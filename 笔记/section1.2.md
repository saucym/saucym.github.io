# 2. Wrap控件
```Dart
Wrap(
        spacing: space,
        runSpacing: space,
        alignment: WrapAlignment.center,
        runAlignment: WrapAlignment.center,
        verticalDirection: VerticalDirection.up,
        textDirection: TextDirection.rtl,
        children: avatars.reversed
            .map((e) => DUIAvatar(
                  url: e,
                  radius: 2,
                  width: w,
                  height: h,
                  loadingWidget: loadingWidget,
                  failedWidget: failedWidget,
                ))
            .toList(),
      )
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
1. `Flutter`里面如何`push`一个类似`alert`的页面，让透明的地方的点击响应传给上一个页面
