title: UIScrollView无限循环的两种方式
date: 2015-10-04 19:55:25
tags:
- 移动开发
- IOS
---

![Alt text](/assets/blogImg/uiscollview_1.png)

## 前言
UIScrollView的无限循环主要指常见的banner图可以左右无限循环滚动，常见的思路为：当我们滑动到最左边第一张的时候，在其左边添加一张UIScrollView，其为图片数组中的最后一位元素，当我们滑动到右边最后一张的时候，在其最右边同样也添加一张UIImageView，其内容为图片数组中的第一位元素，然后当结束滑动时候，重新通过计算当前要展示的页数，设置UIScrollView的偏移量即可；

<!-- more -->

以为为主要代码，当我们初始化图片数组的时候，即在其的头尾将原数组中的最后一位及第一位元素添加在数组中，获取到新数组。

``` objc

-(instancetype)init
{
    self = [super initWithCollectionViewLayout:[self flowLayout]];
    if (self)
    {
        self.imagesArr = [@[@"CycleUIScrollView_0",@"CycleUIScrollView_1",@"CycleUIScrollView_2",@"CycleUIScrollView_3",@"CycleUIScrollView_4"] mutableCopy];
        //将最后一个元素添加在数组头部
        [self.imagesArr insertObject:[self.imagesArr lastObject] atIndex:0];
        //将原数组0  也就是新数组的第一个元素到新数组里
        [self.imagesArr addObject:self.imagesArr[1]];
    }
    return self;
}

```

通过UIScrollview的代理事件获取当前滑动的页数，当达到第一页和最后一页的时候即可重新设置UIScrollview的偏移量，达到一种无限循环的假象。

``` objc
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    if (scrollView == self.collectionView)
    {
        NSInteger page = [self currentPageIndexOffset:scrollView.contentOffset];
        //滑动到第一页
        if (page == 0)
        {
            [self collectionViewScrollToPage:self.imagesArr.count - 2];
        }
        //滑动到最后一页
        if (page == self.imagesArr.count - 1)
        {
            [self collectionViewScrollToPage:1];
        }
    }
}

-(NSInteger)currentPageIndexOffset:(CGPoint)offset
{
    return offset.x / SCREEN_WIDTH;
}

-(void)collectionViewScrollToPage:(NSInteger)page
{
    CGPoint offset = CGPointMake(page * SCREEN_WIDTH, 0);
    self.collectionView.contentOffset = offset;
}
``` 
在这里我使用的是一个UICollectionView，并没有直接使用UIScrollView，由于数据源全部交给了UICollectionView代理事件去处理，所以可以更直观的看到无限循环的效果。在使用UICollectionView要注意设置其UICollectionViewLayout

``` objc
-(UICollectionViewFlowLayout *)flowLayout
{
    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
    layout.minimumInteritemSpacing = 0.f;
    layout.minimumLineSpacing = 0.f;
    layout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
    layout.itemSize = CGSizeMake(SCREEN_WIDTH, SCREEN_WIDTH * 9.0f / 16.f);
    return layout;
}
``` 

### 使用3张UIImageView达到UIScrollview无限循环的目的
这种方法其实是上面的方法的一种变种，但是由于其只创建了3个UIImageView，考虑到了UIImageView的重用，否则当我们的图片数组过多时候会加大内存的消耗。

这种方法重点除了上面方法要考虑滑动到第一张和第三张的时候要重新设置UIScrollView的偏移量，使其重新滑到第二张UIImageView，还有一个地方就是要考虑UIImageView复用的时候图片数组元素的越界问题。

``` objc
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    self.startOffsetX = scrollView.contentOffset.x;
}

- (void)scrollViewWillBeginDecelerating:(UIScrollView *)scrollView
{
    self.endOffsetX = scrollView.contentOffset.x;
}
/**
 *  如果滑到距离过小，则直接返回，否则切换图片的时候循环效果看起来不会太真实
 */
- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    if (fabs(self.startOffsetX - self.endOffsetX) < SCREEN_WIDTH / 2.f)
    {
        return;
    }
    [self scrollViewOffsetXWithPage:1];
    if (self.scrollView == scrollView)
    {
        //向左滑动
        if (self.endOffsetX > self.startOffsetX)
        {
            ((UIImageView *)[self.imageViewArr firstObject]).image = BHIMG(self.imagesArr[self.middlePosition]);
            self.middlePosition++;
            if (self.middlePosition == self.imagesArr.count)
            {
                self.middlePosition = 0;
                ((UIImageView *)[self.imageViewArr lastObject]).image = BHIMG(self.imagesArr[self.middlePosition + 1]);
            }
            else if (self.middlePosition + 1== self.imagesArr.count)
            {
                ((UIImageView *)[self.imageViewArr lastObject]).image = BHIMG(self.imagesArr[0]);
            }
            else
            {
                ((UIImageView *)[self.imageViewArr lastObject]).image = BHIMG(self.imagesArr[self.middlePosition + 1]);
            }
            ((UIImageView *)self.imageViewArr[1]).image = BHIMG(self.imagesArr[self.middlePosition]);
        }
        //向右滑动
        if(self.endOffsetX < self.startOffsetX)
        {
            ((UIImageView *)[self.imageViewArr lastObject]).image = BHIMG(self.imagesArr[self.middlePosition]);
            self.middlePosition--;
            if (self.middlePosition == -1)
            {
                self.middlePosition = self.imagesArr.count - 1;
                ((UIImageView *)[self.imageViewArr firstObject]).image = BHIMG(self.imagesArr[self.middlePosition - 1]);
            }
            else if (self.middlePosition -1 == -1)
            {
                ((UIImageView *)[self.imageViewArr firstObject]).image = BHIMG([self.imagesArr lastObject]);
            }
            else
            {
                ((UIImageView *)[self.imageViewArr firstObject]).image = BHIMG(self.imagesArr[self.middlePosition - 1]);
            }
            ((UIImageView *)self.imageViewArr[1]).image = BHIMG(self.imagesArr[self.middlePosition]);
        }
    }
}

-(void)scrollViewOffsetXWithPage:(NSInteger)page
{
    CGPoint offset = CGPointMake(page * SCREEN_WIDTH, 0);
    self.scrollView.contentOffset = offset;
}
``` 

代码可以下载GITHUB中[BlogDemo](https://github.com/octopusline/BlogDemo.git)进行查看。

