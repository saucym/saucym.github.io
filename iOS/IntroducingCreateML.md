# Introducing Create ML

>去年加入的CoreML framework大获成功，相信许多开发者都已经尝试在自己的app里加入机器学习的功能了，然而从哪里获取模型的问题一直不太好解决。即使苹果提供了一个python工具 --- CoreML Tool，可以将caffe等模型转成coreML模型，但支持的文件类型有限（后面支持了大多数类型），我尝试过，对于我这种ML领域小白，感觉也不太好用。所以苹果今年提供了一个全新的库---createML，他是swift专属的framework，可以解决获取CoreML模型的难题。

这个topic主要介绍了

>Transfer Learning（迁移学习）
图像识别
文本识别
表格数据推断

通过createML和CoreML，可以只用swift一种语言，在mac上解决创建/训练/评估模型-跑模型-部署到端上运行，并且训练、评估模型都是一两句代码就可以实现。对于iOS开发者来说，引入机器学习的成本可以说非常低了，值得关注。CreateML目前只能训练三种类型的数据：图像、文字和表格数据。对应目前的识别能力就是图像、文本识别和根据表格数据推断等。

## Transfer Learning
我准备开始训练数据，然而难题出现了，我上哪找大量用来训练的数据？
得益于transfer learning，仅仅需要少量数据就足够了。由于苹果本身在端上已经内置了多个自己的识别模型（应该是一些大文件，苹果训练了多年的数据），我们训练的模型是基于苹果模型的一种增强，具体就是将他的模型最后几层layer重新根据我们的数据训练。最终做推断时会结合苹果的训练结果和你提供的数据的训练结果来推断。

通过createML重新训练苹果模型的最后几层，最终输出一个coreML模型，这样训练时间大大减少，甚至几秒钟就能训练完成。模型大小也能从100MB级别减少到KB级别（这里我试了下，用Create ML训练的简单模型普遍只有几十kb，用基于python的Turi Create训练出来的动不动几百MB）。

transfer learning让模型可以复用，个人开发者可以轻松训练出体积小的模型，让app更加智能。同时体积小的文件方便下发到端上，甚至内置在安装包里发布。

图像
首先要自己将图像数据分类好，准备好用来训练的数据和用来评估的数据。下图是一种分类方式，将每种类型的图片放在一个文件夹里。然后创建一个swift playground


注意：需要macOS mojave 10.14Beta，Xcode10Beta，并且playground项目最好放在桌面，多试几次import CreateML才能出来。

两行代码，run，就出现一个可视化的窗口
png或jpg文件都可以，尺寸没有要求，把TestImages文件夹和TrainImages文件夹分别拖进来就可以了，当然也可以用纯代码的方式训练，流程是：

指定数据源 - 创建模型 - 评估模型 - 保存模型

如下图。

纯代码训练和评估模型，#!/usr/bin/swift变成可执行文件
这里我使用了https://github.com/Rubenfer/CreateML里提供的图片，故意使用8张图片来训练，只有猫和狗两种类型，19张图片用来评估预测，准确率居然100%，模型文件只有17KB。
然后我将dog文件夹名字改成cat，cat改成dog，发现识别准确率依然100%，对于错误分类的数据也能“正确”的预测出来，当然，你给一张猫狗以外的图片肯定识别错，因为结果枚举里只有猫狗这两种。不过transfer learning还是很牛逼了。

如果对模型满意，只要将模型文件拖到项目里即可，具体CoreML文件的使用参考去年的https://developer.apple.com/videos/play/wwdc2017/703

建议自己尝试https://github.com/Rubenfer/CreateML

文本
data source的格式支持以文件夹分类的txt文本、csv文件和json格式的文件。

文本识别的流程被极大简化了，识别前的语言预测和文本分段都不需要考虑。流程和图片识别类似，只是将MLImageClassifier换成了MLTextClassifier，如下图。

文件夹作为数据源的NLP workflow
json文件作为数据源，使用MLDataTable
虽然今年NLP支持了中文的词性识别和中文机构识别，但我在testData里加入中文好像导致死循环了，不太确定中文数据能否用在createML里。
关于NLP的使用，以前是用NSLinguisticTagger，今年有了一整套新的NL开头的api，参考https://developer.apple.com/videos/play/wwdc2018/713/

建议自己尝试https://github.com/Flight-School/Programming-Language-Classifier，可以获得一个能够识别多种编程语言的模型文件。
识别准确率高达99.64%
表格（TabularData）
createML使用MLDataTable来处理表格数据，datasource支持csv、json，它是基于Turi的。

表格的行是一个example，列是一个feature，通常选取一个列作为target来预测，以example为单位挖掘每个feature之间的关系。这里把price作为target，评估数据时target列的数据对于模型是隐藏的，通过挖掘到的关系来对price做一个inference（预测），再和真实数据里的price比较，评估准确率。


可以方便地筛选想要的example
流程和上面的类似。randomSplit可以将你提供的数据源随机按比例划分成训练数据和评估数据。这里返回的元组，80%的数据是用来训练的，20%是用来评估的。table数据的训练有很多训练器，包括MLBoostedTreeRegressor、MLDecisionTreeRegressor、MLRandomForestRegressor、MLLinearRegressor等，如果不知道要用哪个，可以直接用MLRegressor，他会自动选择一个最合适的。

TabularData workflow
总结
我的理解：createML训练出来的模型必然是依赖苹果模型的，其实本质是用苹果的模型来识别，所以离开苹果的环境应该无法使用。而我们日常开发中费很大力气训练出一个可用的模型，是希望它能运行在多平台的，这样看来CreateML最终产出的CoreML只支持iOS/macOS，在这点上有很大局限性，所以这可能是苹果接下来要解决的另一个难题。
能否让CoreML运行在android等其他平台，将成为CoreML普及的一个关键，目前来看只有纯粹的开发iOS/macOS平台的个人开发者会使用CoreML和CreateML。
虽然如此，我们还是应该开始尝试，毕竟ML的准入门槛已经降低了很多，以后ML也必将成为每个app的标配。



## 这里给一个简单例子
遇到问题：我想把云盘里面的视频和电视剧分类一下
>电影和电视下载太费时间了，可不可以通过名字来分类呢？

收集数据：简单收集公司影音博物馆里面的电影名字和电视剧名字
>要找一个可以快速获取电影和电视名字的地方，一下就想到了影音博物馆

然后训练和评估以及导出模型交给Create ML
```swift
let videoExts = Set(arrayLiteral: "MP4", "MOV", "MKV", "3GP", "MPV", "AVI", "RMVB", "WMF", "MPG", "RM", "ASF", "MPEG", "WMV", "FLV", "F4A", "WEBM", "VOB", "M4V")

let path = "/Users/saucymqin/Desktop/a.txt"
let fileText = try! String(contentsOf: URL(fileURLWithPath: path))
let text = (fileText as NSString).replacingOccurrences(of: "\\", with: "/")

let array = text.components(separatedBy: "\n")
let movies = array.filter { $0.contains("讯电影协会-影音博物馆/★影视/电影/") }
let tvs = array.filter { $0.contains("讯电影协会-影音博物馆/★影视/剧集/电视剧") }
let movieNames = movies.filter { videoExts.contains(($0 as NSString).pathExtension.uppercased()) }.map{ ($0 as NSString).lastPathComponent }

let tvNames = tvs.filter { videoExts.contains(($0 as NSString).pathExtension.uppercased()) }.map{ ($0 as NSString).lastPathComponent }

print("movie count:\(movieNames.count)  tv count: \(tvNames.count)")

print("Specify Data Create Model")
let classifier = try MLTextClassifier(trainingData: ["movie": movieNames, "tv": tvNames])
let evaluation = classifier.evaluation(on: ["movie": ["复仇者联盟", "大话西游2"], "tv": ["延禧攻略.2018.720p.X264.第34集", "延禧攻略.2018.720p.X264.第36集"]])
print("Evaluate Model \n\(evaluation.description)")
print("Save Model")
try classifier.write(toFile: "/Users/saucymqin/Desktop/Test/VideoClassifier.mlmodel")
```
