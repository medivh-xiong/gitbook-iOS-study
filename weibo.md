# 微博个人页的实现

## 一、分析项目的结构
* **效果：当用户滑动时候的滑到一定距离时候出现导航栏**
* **结论：肯定是``导航控制器``，只不过先隐藏导航栏，通过监听滑动的代理事件，动态的改变导航栏的透明度**
* **效果：往下拉动的时候，会让最上面的图片放大，往上滑动时候，图片从下半部分开始被裁剪，同时导航栏渐变出来，同时下半部分是图文信息**
* **结论：图文信息部分应该是```tableView```设置的，至于上面的图片和头像部分可能是有```headView```也有可能是```view```实现的**


##二、功能的层次结构
1. 首先发先往上拉动tableview可以穿透导航栏  
  * ``tableView``：尺寸应该是整个屏幕，否则无法穿透导航栏；
2. ``tableView``是全屏的，但是又有导航栏。
  * 一开始先把导航栏设置透明，``导航栏默认颜色是一张图片，下面的横线阴影也是一张图片``。
3. ``tableView``的导航栏是默认是不显示内容的，然后渐变出文字内容。
  * 通过修改title发现没法改变透明度，因此可以设置```titleView```用一个label来表示中间的标题栏。 
4. 滑动到一个导航栏的距离时候标签页出现悬停效果。
  * ``tableView``的``headView``无法实现悬停效果，因此标签页是一个``View``现实，同时上面的图片的``View``也是一个``View``上面放一个``imageView``。
5. 滑动时候上方的图片会往上一起滑动到一定程度。
  *``view``可以设置约束，通过监听的``scrollView``的滚动代理方法，来改变约束。
  
  
##三、功能的实现

###1.搭建界面
根据层次结构，可以发现这个界面是一个导航控制器所包裹的``ViewController``界面，同时有3个界面构成，首先是一个头部视图，然后是一个可以切换标签的视图，最后下面是一个``tableView``。

>     // ----苹果官方自动回做一个处理，会在导航控制的上面自动添加导航栏，并且其他空间高度自动产生变化，这个变化是在ViewDidAppear这个方法里面添加的，因此要做一个处理；
    self.automaticallyAdjustsScrollViewInsets = NO;

设置这个方法来取消官方对导航栏高度的自动处理，这样就可以让``tableView``全屏了。
1. 设置``tableView的frame和屏幕尺寸一致``，把``tableView添加到一个VC上``
2. ``在VC上添加图片视图和标签按钮视图``，这2个视图和tablview是平级关系，如果代码创建，要把这2个视图移到最前面。
3. 设定图片视图和标签视图的约束
4. 设定导航控制的透明以及字体的透明 

>  // ----然后隐藏导航栏,苹果官方做了处理了，如果给任意图片都会做处理，因此传递一个空的image就会变成透明，类型一定要改成defualt否则无法设置背景图片
    [self.navigationController.navigationBar setBackgroundImage:[[UIImage alloc] init] forBarMetrics:UIBarMetricsDefault];
   
    [self.navigationController.navigationBar setShadowImage:[[UIImage alloc] init]];
    
    UILabel *titleLabel = [[UILabel alloc] init];
    titleLabel.text = @"我是熊欣";
    
    [titleLabel setTextColor:[UIColor colorWithWhite:1 alpha:0]];
    [titleLabel sizeToFit];
    
    // ----设置titleView
    self.navigationItem.titleView = titleLabel;


### 2.逻辑处理
1.界面搭好之后，发现tableView的内容被遮挡了，这时候就需要设置tableView的contentInset来显示内容，空出上面顶部视图,因此需要设置空出顶部视图高度加上按钮视图的高度
 
> self.contentTable.contentInset = UIEdgeInsetsMake(headViewHeight + btnViewHeight, 0, 0, 0);

2.这时候发现拖动只有tablview的表格在变化，顶部视图没有变化，这时候就需要监听滚动的代理方法，当滚动时候，改变顶部视图的约束，最完美的做法是改变顶部图片视图的高度，而顶部视图高度：原始高度-向上偏移的高度，而向上偏移的高度= 移动后的表格的y - 初始的表格的y

>   // ----移动的高度=移动后的y-最初的y
    CGFloat scrollHeight = scrollView.contentOffset.y - self.originOffsetY;
    
**初始的表格的y不能直接通过self.contentTable.contentOffset.y获得**

>     // ----tableView的偏移量是tablview内容与可视范围的差值，没有设置contentInset就没有偏移量
//        self.originOffsetY = self.contentTable.contentOffset.y;
    self.originOffsetY = - (XHheadViewHeight + XHbtnViewHeight);

**注意点：当调用``contentInset``方法的时候，会自动调用``-(void)scrollViewDidScroll:(UIScrollView *)scrollView``这个方法。而设置``contentInset``后``contentOffset``也会发生变化，因为本质就是``tablview内容与可视范围的差值``,因此这边要存储最开始的``contentOffset.y``要你之前偏移的高度的具体数值，而不能直接取``self.contentTable.contentOffset.y``,同时要记住用成员变量存开始的偏移值时候，要放在设置``contentInset``之前，否则会先调用滚动的代理方法**

然后设置顶部视图的高度为拖动后的高度：

>   CGFloat h = XHheadViewHeight - scrollHeight;

通过观察发现，滑动时候有一个悬停效果，当距离为导航栏距离的时候，就停止缩小，然后显示出导航栏，这里要对h进行判断，如果h<导航栏高度的时候，就让它=导航栏的高度。

> // ----添加悬停效果
    if (h <= XHheadViewMinH) {
        h = XHheadViewMinH;
    }

 最后设置顶部视图的高度为h：
 
>  self.topViewHeight.constant = h;

然后设定导航栏字体和颜色渐变->在滚动的时候动态的改变``alpha``就可以实现了，**注意点：导航栏颜色不能改变，因此需要采用图片显示，这样可以写一个image的分类根据传过去的颜色生成image对象，来生成，但是系统导航栏当图片透明度为1，然后又覆盖系统官网导航栏图片情况下，会做处理，让导航栏变透明，因此当alpha大于1时候手动调整成0.99**

> CGFloat alpha = scrollHeight / (XHheadViewHeight - XHheadViewMinH);
   
>  if (alpha >= 1) {
        alpha = 0.99f;
    }

>  self.titleLabel.textColor = [UIColor colorWithWhite:0 alpha:alpha];

>  // ----设定导航栏的渐变色，因为导航栏颜色没法设置透明度，只能从图片进行修改
   UIImage *image = [UIImage imageWithColor:[UIColor colorWithWhite:1 alpha:alpha]];
    
>  [self.navigationController.navigationBar setBackgroundImage:image forBarMetrics:UIBarMetricsDefault];


### 源码
https://github.com/medivh-xiong/-.git









