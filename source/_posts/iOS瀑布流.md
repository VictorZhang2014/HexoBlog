---
title: iOS瀑布流
date: 2017-11-11 18:33:36
tags: 瀑布流, UICollectionView
categories: iOS
---
<img src="/img/iOS/autolayout/WaterFallFlowLayout_1.jpg" alt="" width="300" height="500" /><img src="/img/iOS/autolayout/WaterFallFlowLayout_2.jpg" alt="" width="300" height="500" />
<a href="https://github.com/VictorZhang2014/WaterFallFlowLayout" target="_blank">Demo地址</a>

## 一、实现方式
- 1.`UIScrollView`        重点：`视图重用`
- 2.`UITableView`         重点：`滑动同步`
- 3.`UICollectionView`    重点：`布局`

## 二、实现代码
本文以`UICollectionView`作为讲解，要做成`瀑布流`的效果，其实很简单；基本分为三个步骤
- 1.创建一个基本的`UICollectionView`，但是cell的大小不一致
- 2.自定义一个类派生自`UICollectionViewFlowLayout`
- 3.在自定义的的flowLayout类里，实现：
    - 3.1 获取`UICollectionView`的所有cell
    - 3.2 对所有的`cell`重新计算`frame`
    - 3.3 将重新计算的`frame`赋值给`cell`

代码1，创建`UICollectionView`
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    WaterFallFlowLayout *flowLayout = [[WaterFallFlowLayout alloc] init];
    self.collectionView = [[UICollectionView alloc] initWithFrame:self.view.bounds collectionViewLayout:flowLayout];
    self.collectionView.backgroundColor = [UIColor yellowColor];
    self.collectionView.delegate = self;
    self.collectionView.dataSource = self;
    //注册单元格
    [self.collectionView registerClass:[WaterFallCollectionViewCell class] forCellWithReuseIdentifier:identifier];
    [self.view addSubview:self.collectionView];
}

#pragma mark - UICollectionView dataSource
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return self.imgArr.count;
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    WaterFallCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:identifier forIndexPath:indexPath];
    if (!cell) {
        cell = [[WaterFallCollectionViewCell alloc] init];
    }
    cell.image = self.imgArr[indexPath.item];
    return cell;
}

- (float)imgHeight:(float)height width:(float)width{
    /*
        高度/宽度 = 压缩后高度/压缩后宽度（100）
     */
    float newHeight = height / width * 100;
    return newHeight;
}

#pragma mark - UICollectionView delegate flowLayout
- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath{
    UIImage *image = self.imgArr[indexPath.item];
    float height = [self imgHeight:image.size.height width:image.size.width];
    return CGSizeMake(100, height);
}

- (UIEdgeInsets)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout insetForSectionAtIndex:(NSInteger)section{
    CGFloat margin = (self.view.frame.size.width - 100 * 3) / 4;
    UIEdgeInsets edgeInsets = {margin,margin,margin,margin};
    return edgeInsets;
}
```

自定义一个类派生自`UICollectionViewFlowLayout`，代码如下
```
@interface WaterFallFlowLayout()

@property (nonatomic, weak) id<UICollectionViewDelegateFlowLayout> delegate;
@property (nonatomic, assign) NSInteger cellCount;//cell的个数
@property (nonatomic, strong) NSMutableArray<NSNumber *> *colArr;//存放列的高度
@property (nonatomic, strong) NSMutableDictionary<NSString *, NSIndexPath *> *attributeDict;//存放cell的位置信息
@property (nonatomic, assign) int colCount;//cell共有几列

@end

@implementation WaterFallFlowLayout

//1.准备布局：
//  1.1 得到cell的总个数，
//  1.2 为每个cell确定自己的frame位置
- (void)prepareLayout{
    [super prepareLayout];
    
    _colCount = 3;
    _colArr = [NSMutableArray<NSNumber *> array];
    _attributeDict = [NSMutableDictionary<NSString *, NSIndexPath *> dictionary];
    self.delegate = (id<UICollectionViewDelegateFlowLayout>)self.collectionView.delegate;
    
   //获取cell的总个数
    _cellCount = [self.collectionView numberOfItemsInSection:0];
    if (_cellCount == 0) {
        return;
    }
    
    //假设一开始cell的列的高度都等于0
    for (int i = 0; i < _colCount; i++) {
        [_colArr addObject:[NSNumber numberWithFloat:0]];
    }
    
    //循环调用layoutItemAtIndexPath方法，为每个cell布局，将indexPath传入，作为布局字典的key
    //layoutAttributesForItemAtIndexPath方法的实现，这里用到了一个布局字典，其实就是将每个cell的位置信息与indexPath相对应，将它们放到字典中，方便后面视图的检索
    for (int i = 0; i < _cellCount; i++) {
        [self layoutItemAtIndexPath:[NSIndexPath indexPathForItem:i inSection:0]];
    }
}

//此方法会多次调用，为每个cell布局
- (void)layoutItemAtIndexPath:(NSIndexPath *)indexPath{
    //通过协议得到cell的间隙
    UIEdgeInsets edgeInsets = [self.delegate collectionView:self.collectionView layout:self insetForSectionAtIndex:indexPath.row];
    
    //得到每个cell的size
    CGSize itemSize = [self.delegate collectionView:self.collectionView layout:self sizeForItemAtIndexPath:indexPath];
    
    float col = 0;
    float shortHeight = [[_colArr objectAtIndex:col] floatValue];
    
    //找出高度最小的列，将cell加到最小列中
    for (int i = 0; i < _colArr.count; i++) {
        float height = [[_colArr objectAtIndex:i] floatValue];
        if (height < shortHeight) {
            shortHeight = height;
            col = i;
        }
    }
    
    float top = [[_colArr objectAtIndex:col] floatValue];
    //确定cell的最终frame
    CGRect frame = CGRectMake(edgeInsets.left + col * (edgeInsets.left + itemSize.width), top + edgeInsets.top, itemSize.width, itemSize.height);
    
    //将列高加入到数组里
    [_colArr replaceObjectAtIndex:col withObject:[NSNumber numberWithFloat:top + edgeInsets.top + itemSize.height]];
    
    //将每个cell的frame对应一个indexPath，加入到字典中
    [_attributeDict setObject:indexPath forKey:NSStringFromCGRect(frame)];
}

//返回cell的布局信息，如果忽略传入的rect一次性将所有的cell布局信息返回，图片过多时性能会很差
- (NSArray *)layoutAttributesForElementsInRect:(CGRect)rect{
    NSMutableArray *muArr = [NSMutableArray array];
    //indexPathsOfItem方法，根据传入的frame值计算当前应该显示的cell
    NSArray *indexPaths = [self indexPathsOfItem:rect];
    for (NSIndexPath *indexPath in indexPaths) {
        UICollectionViewLayoutAttributes *attribute = [self layoutAttributesForItemAtIndexPath:indexPath];
        [muArr addObject:attribute];
    }
    return muArr;
}

//为每个cell布局完毕后，需要实现这个方法， 传入frame，返回的时cell的信息
//传入当前可见cell的rect，视图滑动时调用
- (NSArray *)indexPathsOfItem:(CGRect)rect{
    //遍历布局字典通过CGRectIntersectsRect方法确定每个cell的rect与传入的rect是否有交集，如果结果为true，则此cell应该显示，将布局字典中对应的indexPath加入数组
    NSMutableArray *array = [NSMutableArray array];
    for (NSString *rectStr in _attributeDict) {
        CGRect cellRect = CGRectFromString(rectStr);
        if (CGRectIntersectsRect(cellRect, rect)) {
            NSIndexPath *indexPath = _attributeDict[rectStr];
            [array addObject:indexPath];
        }
    }
    return array;
}

//把重新计算的frame赋值给cell
- (UICollectionViewLayoutAttributes*)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath{
    UICollectionViewLayoutAttributes *attributes = [UICollectionViewLayoutAttributes layoutAttributesForCellWithIndexPath:indexPath];
    for (NSString *rectStr in _attributeDict) {
        if ([indexPath compare:_attributeDict[rectStr]] == NSOrderedSame) {
            attributes.frame = CGRectFromString(rectStr);
        }
    }
    return attributes;
}

//最后还要实现这个方法，返回collectionView内容的大小
//只需要遍历前面创建的存放列高的数组得到列最高的一个作为高度返回就可以了
- (CGSize)collectionViewContentSize{
    CGSize size = self.collectionView.frame.size;
    float maxHeight = [[_colArr objectAtIndex:0] floatValue];
    //查找最高的列的高度
    for (int i = 0; i < _colArr.count; i++) {
        float colHeight = [[_colArr objectAtIndex:i] floatValue];
        if (colHeight > maxHeight) {
            maxHeight = colHeight;
        }
    }
    size.height = maxHeight;
    return size;
}

@end

```

<a href="https://github.com/VictorZhang2014/WaterFallFlowLayout" target="_blank">Demo地址</a>
