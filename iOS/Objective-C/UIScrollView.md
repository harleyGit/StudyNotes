- **UIScrollView优化**
- **UIScrollView控制手势**
- **UIScrollView重用子视图**


<br/>

***
<br/>

># UIScrollView优化

**内存图片的托管**

&emsp;  将scrollview上所有的图片指针收集起来，当图片占用内存到达某个量后，用`setImage: nil`将可视区域之外的图片全部删除，这个方法基本上能够解决图片加载过多的crash问题，但是由于承载image的view没有被删除，长时间滑动也会出现抖动的现象。

<br/>

- **重用UIScrollView**

&emsp;模仿UITableView的实现机制，将scrollview进行重用，当scrollview上的view滑出可视范围是，就将滑出的view重新设置rect添加到scrollview的最后，这样的实现方式，保证在scrollview从开始到结束只有渲染一屏，多屏只是重新reload资源，从而让海量图片浏览也能保持高性能。

&emsp;UIScrollView是iPhone中的一个重要的视图，它提供了一个方法，让你在一个界面中看到所有的内容，从而不必担心因为屏幕的大小有限，必须翻到下一页进行阅览。确实对于用户来说是一个很好的体验。但是又是如何把所有的内容都加入到scrollview，是简单的addsubView。假如是这样，岂不是scrollView界面上要放置很多的图形，图片。移动设备的显示设备肯定不如PC，怎么可能放得下如此多的视图。所以在使用scrollView中一定要考虑这个问题，当某些视图滚动出可见范围的时候，应该怎么处理，是不管它那，还是进行内存回收或者重利用。苹果公司的UITableView就很好的展示了在UIScrollView中如何重用可视的空间，减少内存的开销。


<br/>

- **官方解释UIScrollView**

&emsp;首先还是看官方API如何解释UIScrollView，翻译官方UIScrollView的帮助文档。

&emsp;   UIScrollView类支持显示比屏幕更大的应用窗口的内容。它通过挥动手势，能够使用户滚动内容，并且通过捏合手势缩放部分内容。

&emsp;  `UIScrollView是UITableView和UITextView的超类。`

&emsp;  UIScrollView的核心理念是，它是一个可以在内容视图之上，调整自己原点位置的视图。它根据自身框架的大小，剪切视图中的内容，通常框架是和应用程序窗口一样大。一个滚动的视图可以根据手指的移动，调整原点的位置。展示内容的视图，根据滚动视图的原点位置，开始绘制视图的内容，这个`原点位置`就是滚动视图的`偏移量`。ScrollView本身不能绘制，除非显示水平和竖直的指示器。滚动视图必须知道内容视图的大小，以便于知道什么时候停止；一般而言，当滚动出内容的边界时，它就返回了。

&emsp;  某些对象是用来管理内容显示如何绘制的，这些对象应该是管理如何平铺显示内容的子视图，以便于没有子视图可以超过屏幕的尺寸。就是当用户滚动时，这些对象应该恰当的增加或者移除子视图。

&emsp;  因为滚动视图没有滚动条，它必须知道一个触摸信号是`打算滚动`还是打算`跟踪里面的子视图`。为了达到这个目的，它临时中断了一个touch-down的事件，通过建立一个定时器，在定时器开始行动之前，看是否触摸的手指做了任何的移动。假如定时器行动时，没有任何的大的位置改变，滚动视图就发送一个跟踪事件给触摸的子视图。如果在定时器消失前，用户拖动他们的手指足够的远，滚动视图取消子视图的任何跟踪事件，滚动它自己。子类可以重载`touchesShouldBegin:withEvent:inContentView:`, `pagingEnabled`,和`touchesShouldCancelInContentView:` 方法，从而影响滚动视图的滚动手势。

&emsp;  一个滚动视图也可以控制一个视图的缩放和平铺。当用户做捏合手势时，滚动视图调整偏移量和视图的比例。当手势结束的时候，管理视图内容显示的对象，就应该恰当的升级子视图的显示。当手势在处理的过程中，滚动视图不能够给子视图，发送任何跟踪的调用。

&emsp;  UIScrollView类有一个delegate，需要适配的协议是`UIScrollViewDelegate`。为了缩放和平铺工作，代理必须实现`viewForZoomingInScrollView:`和 `scrollViewDidEndZooming:withView:atScale:` 方法。另外，最大和最小缩放比例应该是不同的。

&emsp;  重要的提示：在UIScrollView对象中，你`不应该嵌入任何UIWebView和UITableView`。假如这样做，会出现一些异常情况，因为2个对象的触摸事件可能被混合，从而错误的处理(因为滚动收视造成混淆，可以通过其他的办法进行[**解决**](https://www.cnblogs.com/6duxz/p/10044873.html))。

<br/>

***
<br/>


># UIScrollView控制手势

&emsp;  这些都是官方API的解释，重点是理解UIScrollView怎么来控制手势的。可以由`canCancelContentTouches`这个方法的运用来解释UIScrollView如何控制手势的。

&emsp;  假如你设置`canCancelContentTouches`为YES，那么当你在UIScrollView上面放置任何子视图的时候,当你在子视图上移动手指的时候，UIScrollView会给子视图发送`touchCancel的消息`。而如果该属性设置为NO，ScrollView本身不处理这个消息，全部交给子视图处理。

&emsp;  那么这里就有疑问了，既然该属性设置未来NO了，那么岂不是UIScrollView不能处理任何事件了，那么为何在子视图上快速滚动的时候，UIScrollView还能移动那。这个一定要区分前面所说的UIScrollView中断touch-Down事件，开启一个定时器。我们设置的这个`cancancelContentTouches` 属性为NO时，只是让UIScrollView不能发送cancel事件给子视图。而前面所说中断touch-down事件，和取消touch事件是俩码事，所以当快速在子视图上移动的时候，当然可以滚动。但是如果你慢速的移动的话，就可以区分这个属性了，假如设定为YES，在子视图上慢速移动也可以滚动视图，但是如果为NO 。因为UIScrollView，发送了cancel事件给子视图处理了，自己当然滚动不了了。




<br/>

***
<br/>


># UIScrollView重用子视图

&emsp;  事件处理看过了，就要考虑scrollView如何重用内存的，下面写了一个例子模仿UITableView的重用的思想，这里只是模仿，至于苹果公司怎么实现这种重用的，他们应该有更好的方法。

&emsp;  这里的例子是在scrollView上放置4个2排2列的视图，但是内存中只占用6个视图的内存空间。当scrollView滚动的时候，通过不停的重用之前视图的内存空间，从而达到节省内存的效果。重用的方法如下：

>a.如果scrollView向下面滚动，一旦一排视图滚出了可视范围，就改变滚动出去的那个view在scrollView中的frame，也就是改变位置到达末尾，达到重用的效果。

>b.如果scrollView向上面滚动，一旦最末排的视图view滚出了可视范围，就改变滚动出去的那个view在scrollView中的frame，移动到最前面。


&emsp;  下面就需要在你创建的视图控制器中，创建一个重用的视图数组，用来把这些要显示的视图放入内存中，这里虽然界面上显示的是2排2列的四个视图，但是当拖动的时候，可能出现前面一排的视图显示一部分，末尾一排的视图显示一部分的情况，所以重用的数组中要放置6个视图。下面是定义的一些宏：

```
#define sMyViewTotal 6
#define sMyViewWidth 150
#define sMyViewHeight 220
#define sMyViewGap 10


_aryViews = [[NSMutableArray alloc] init];

for (int i = 0; i < sMyViewTotal; i++) {
    CGFloat x;
    if (i%2) {
        x = sMyViewWidth + sMyViewGap + sMyViewGap/2;
    }else{
        x = sMyViewGap/2;
    }

    CGFloat y = (sMyViewHeight + sMyViewGap)*(i/2);

    MyView *myView = [[MyView alloc] initWithFrame:CGRectMake(x, y, sMyViewWidth, sMyViewHeight)];
    myView.showNumber = i;
   [myScrollView addSubview:myView];
   [_aryViews addObject:myView];
}

```


<br/>

***所以这里的核心方法是，首先要判断是向上滚动还是向下滚动方法如下：***

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{

    BOOL directDown;
    if (previousOffSet.y < scrollView.contentOffset.y) {
        directDown = YES;
    } else{
        directDown = NO;
    }

    previousOffSet.y = scrollView.contentOffset.y;

    //防止最开始就向上面拖动的时候，改变数组视图树的位置。
    if (scrollView.contentOffset.y < 0) {
        return;
    }

    if (directDown) {
        NSLog(@"down");
        MyView * subView = [_aryViews objectAtIndex:firstViewIndex];
        CGFloat firstViewYOffset = subView.frame.origin.y + subView.frame.size.height + sMyViewGap;

        //寻找第一个视图是否滚动出去
        if (firstViewYOffset < scrollView.contentOffset.y) {
            //改变数组中第一排可见视图的位置。
            [self moveIndexInViewsWithDirect:YES];
        }

    }else{

        NSLog(@"up");
        MyView * subView = [_aryViews objectAtIndex:(firstViewIndex + sMyViewTotal - 2)%sMyViewTotal];

        CGFloat lastViewYOffset = subView.frame.origin.y - scrollView.bounds.size.height;
        if (lastViewYOffset > scrollView.contentOffset.y) {
              [self moveIndexInViewsWithDirect:NO];
        }
    }

}
```
&emsp;  每次滚动的时候先判断滚动位置即offset，和先前的比较。如果先前的大就是向下滚动，否则就是向上滚动。

&emsp;  找到了向下滚动了，就该判断是否子视图已经离开了可视范围。方法就是判断当前offset和视图的位置进行比较。如果判断滚到离开了可视范围，然后就是要改变重用视图数组中第一个视图的位置了。这里用了firstViewIndex来记录scrollView中第一个可见视图的位置， 循环使用这6个视图达到重用的目的。自然firstViewIndex上面的一个视图就是最后一个视图的位置（firstViewIndex + sMyViewTotal - 1）％sMyViewTotal。所以这里需要改变重用视图中firstViewIndex即第一个可见视图的位置。代码如下：

```
- (void)moveIndexInViewsWithDirect:(BOOL)forward{

    [UIView setAnimationsEnabled:NO];

    if (forward) {

        for (int i = firstViewIndex; i < (firstViewIndex + 2); i++) {

            MyView *subView = [_aryViews objectAtIndex:i%sMyViewTotal];
            subView.showNumber = subView.showNumber + sMyViewTotal;

            subView.frame = CGRectMake(subView.frame.origin.x, subView.frame.origin.y + (sMyViewTotal/2) * (sMyViewHeight + sMyViewGap), subView.frame.size.width, subView.frame.size.height);

        }

        firstViewIndex = (firstViewIndex + 2)%sMyViewTotal;

    }else{

        int lastViewIndex = firstViewIndex + sMyViewTotal - 1;
        for (int i = lastViewIndex; i > (lastViewIndex - 2); i--) {           
             MyView *subView = [_aryViews objectAtIndex:(firstViewIndex + sMyViewTotal - i)%sMyViewTotal];

             subView.showNumber = subView.showNumber - sMyViewTotal;

            subView.frame = CGRectMake(subView.frame.origin.x, subView.frame.origin.y - (sMyViewTotal/2) * (sMyViewHeight + sMyViewGap), subView.frame.size.width, subView.frame.size.height);

        }

        firstViewIndex = (firstViewIndex + sMyViewTotal - 2)%sMyViewTotal;

    }

    [UIView setAnimationsEnabled:YES];

}

```

&emsp; 这里创建的子视图数字属性，是用来在视图上画数字的，这样就可以看到视图重用的效果了，应该是从0开始到无穷多，但是实际上内存中就创建了6个视图。

```
- (void)drawRect:(CGRect)rect
{

    // Drawing code

    NSString *text = [NSString stringWithFormat:@"%d",showNumber];

    [[UIColor redColor] set];

    [text drawInRect:CGRectMake(rect.origin.x, rect.origin.y + rect.size.height/2 - 30, rect.size.width, 30) withFont:[UIFont      fontWithName:@"Helvetica" size:20] lineBreakMode:UILineBreakModeWordWrap alignment:UITextAlignmentCenter];

} 
```






















