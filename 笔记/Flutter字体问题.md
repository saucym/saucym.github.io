# 3. 在`flutter`里面`iOS14`英文字体宽度被压缩的问题
`iOS`和`flutter`默认字体都是这个`SFUI`，跟我们说的`苹果方体`还有一些区别，`SF`是`San Francisco`的缩写，是`苹方字体`的英文部分，中文部分叫`PingFang`。
但是，两者之间是有差距的。比如对数字的显示，`PingFang`显示的数字与前后汉字更加紧凑，但是`SFUI`与前后汉字之间会有一定间隔。

[参考`issues`](https://github.com/flutter/flutter/issues/60013)
