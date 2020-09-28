# Create ML实践

Core ML 让你可以轻松将机器学习模型应用于你的 App 中。但是这些模型从哪里来，除了直接网上下载WWDC 2018 - Session 703 Introducing Create ML里面苹果给出了自己的机器学习方案，使用Mac和Swift就可以训练自己的模型。简介可以参考[这篇文章](https://www.jianshu.com/p/2ef5d672ceff)

## 一般机器学习步骤
遇到问题->收集数据->训练->评估->模型

## 这里给一个简单例子
遇到问题：我想把云盘里面的视频和电视剧分类一下
>电影和电视下载太费时间了，可不可以通过名字来分类呢？

收集数据：简单收集公司影音博物馆里面的电影名字和电视剧名字
>要找一个可以快速获取电影和电视名字的地方，一下就想到了影音博物馆

然后训练和评估以及导出模型交给Create ML
```swift
import CreateML

let videoExts = Set(arrayLiteral: "MP4", "MOV", "MKV", "3GP", "MPV", "AVI", "RMVB", "WMF", "MPG", "RM", "ASF", "MPEG", "WMV", "FLV", "F4A", "WEBM", "VOB", "M4V")

let path = "/Users/qin/Desktop/a.txt"
let fileText = try! String(contentsOf: URL(fileURLWithPath: path))
let text = (fileText as NSString).replacingOccurrences(of: "\\", with: "/")

let array = text.components(separatedBy: "\n")
let movies = array.filter { $0.contains("电影协会-影音博物馆/★影视/电影/") }
let tvs = array.filter { $0.contains("电影协会-影音博物馆/★影视/剧集/电视剧") }
let movieNames = movies.filter { videoExts.contains(($0 as NSString).pathExtension.uppercased()) }.map{ ($0 as NSString).lastPathComponent }

let tvNames = tvs.filter { videoExts.contains(($0 as NSString).pathExtension.uppercased()) }.map{ ($0 as NSString).lastPathComponent }

print("movie count:\(movieNames.count)  tv count: \(tvNames.count)")

print("Specify Data Create Model")
let classifier = try MLTextClassifier(trainingData: ["movie": movieNames, "tv": tvNames])
let evaluation = classifier.evaluation(on: ["movie": ["复仇者联盟", "大话西游2"], "tv": ["延禧攻略.2018.720p.X264.第34集", "延禧攻略.2018.720p.X264.第36集"]])
print("Evaluate Model \n\(evaluation.description)")
print("Save Model")
try classifier.write(toFile: "/Users/qin/Desktop/Test/VideoClassifier.mlmodel")
```
