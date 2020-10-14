---
title: 为你的APP加上DragDrop
categories: iOS
tages: [iOS11,DragDrop]
---
苹果最近发布了iOS11beta版，里面有很多激动人心的API，我们这里先体验一下DragDrop。Drag做为定义方提供一个可以拖拽的东西，可以是图片或文件等任意的东西。Drop是作为接收方，你可以指定你可以接收一些什么东西。

我们这里以一个UICollectionView为例子，这里苹果已经给我们实现了这两个协议。
```Objective-C
@property (nonatomic, weak, nullable) id <UICollectionViewDragDelegate> dragDelegate API_AVAILABLE(ios(11.0)) API_UNAVAILABLE(tvos, watchos);
@property (nonatomic, weak, nullable) id <UICollectionViewDropDelegate> dropDelegate API_AVAILABLE(ios(11.0)) API_UNAVAILABLE(tvos, watchos);
```
我们只要做很少的事情就可以搞定了一个简单的DragDrop了
一、首先在你的CollectionViewController里面指定协议
```Objective-C
if (@available(iOS 11.0, *)) {
    self.collectionView.dropDelegate = self;
    self.collectionView.dropDelegate = self;
}
```
二、实现Drop协议
1.首先制定接收拖拽的方式，我们这里指定为copy，这样默认实现就不会移除drag位置的cell
```Objective-C
- (UICollectionViewDropProposal *)collectionView:(UICollectionView *)collectionView dropSessionDidUpdate:(id<UIDropSession>)session withDestinationIndexPath:(nullable NSIndexPath *)destinationIndexPath {
    return [[UICollectionViewDropProposal alloc] initWithDropOperation:UIDropOperationCopy];
}
```
2.指定是否可以接收拖拽的东西，这里可以通过session的方法 hasItemsConformingToTypeIdentifiers: 判断是否是你可以处理的拖拽类型，比如这里只处理拖过来的图片
```Objective-C
- (BOOL)collectionView:(UICollectionView *)collectionView canHandleDropSession:(id<UIDropSession>)session {
    return [session hasItemsConformingToTypeIdentifiers:@[(NSString *)kUTTypeImage]];
}
```
3.处理拖拽过来的数据
```Objective-C
- (void)collectionView:(UICollectionView *)collectionView performDropWithCoordinator:(id<UICollectionViewDropCoordinator>)coordinator {
    if (coordinator.session.localDragSession) { //来自自己当前APP的拖拽,可以直接拿到drag的时候自定义的数据
        CGPoint point = [coordinator.session.localDragSession locationInView:collectionView];
        NSIndexPath *indexPath = [collectionView indexPathForItemAtPoint:point]; //拿到了松手时候的目标cell  可以拿来做动画
        
        [coordinator.items enumerateObjectsUsingBlock:^(id<UICollectionViewDropItem>  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            UIDragItem *dragItem = obj.dragItem;
            if (dragItem.localObject) {
                id object = dragItem.localObject; //这里就是drag的时候传的自定义数据
            }
        }];
    } else { // 来自别的APP的拖拽，这里需要从别的APP下载数据
        [coordinator.items enumerateObjectsUsingBlock:^(id<UICollectionViewDropItem>  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            UIDragItem *dragItem = obj.dragItem;
            [dragItem.itemProvider loadFileRepresentationForTypeIdentifier:(NSString*)kUTTypeJPEG completionHandler:^(NSURL * _Nullable url, NSError * _Nullable error) {
                //下载到拖拽过来的数据了
            }];
        }];
    }
}
```
三、实现Drag协议
1.定制拖拽的对象，需要实现一个NSItemProviderWriting协议，告诉目标APP我们拖的是一个什么类型的数据，然后把数据处理一下给目标APP，还可以给一个处理进度给目标APP
```Objective-C
@interface WYDragFileItem : NSObject <NSItemProviderWriting>
@end

@implementation WYDragFileItem

+ (NSArray<NSString *>*)writableTypeIdentifiersForItemProvider {
    return @[(NSString*)kUTTypeJPEG, (NSString *)kUTTypeImage];
}

- (nullable NSProgress *)loadDataWithTypeIdentifier:(NSString *)typeIdentifier forItemProviderCompletionHandler:(void (^)(NSData * _Nullable data, NSError * _Nullable error))completionHandler { //这里面可以是一个下载方法或者压缩方法或者格式转换方法，允许给一个进度让目标位置显示，可异步
    completionHandler(UIImagePNGRepresentation([UIImage imageNamed:@"image"]), nil);
    return nil;
}

@end
```
2.开始拖拽的时候包装拖拽数据
```Objective-C
- (NSArray<UIDragItem *> *)collectionView:(UICollectionView *)collectionView itemsForBeginningDragSession:(id<UIDragSession>)session atIndexPath:(NSIndexPath *)indexPath {
    NSItemProvider *provider = [[NSItemProvider alloc] initWithObject:dragObject]; //dragObject 是一个实现NSItemProviderWriting协议的数据，一会下面介绍
    UIDragItem *item = [[UIDragItem alloc] initWithItemProvider:provider];
    item.localObject = anyObject; //anyObject 是可以传递的任意自定义数据
    return @[item];
}
```
    
这样一个简单的拖拽流程就完成了，细节或动画等这里就不写了，具体可以相互探讨一下。
