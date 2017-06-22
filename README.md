## React Native实战总结

入职有道数月，主要参与了精品课垂直页的改版，期间遇到了不少坑，虽然还未正式上线，但是也值得总结一波，故写下此文。

### 需求介绍

*  首页部分：加入日志控制工具以收集、统计用户操作；改写搜索框写法，将搜索框提取成独立的共用组件。
*  垂直页部分：此次改版集中在垂直页，新版垂直页将导航部分改版加入搜索框；重新设计了Banner部分；课程分类重新设计；课程入口信息增多样式改变；将原本的两列布局改成新版的一列布局，还加入了原本没有的（图片）小标题。

### 实现方案及问题

#### 一、日志控制

###### 首页布局

<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/home1.jpg' align='left' width='31%'>

<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/home2.jpg' align='left' width='31%'>

<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/home3.jpg' align='left' width='31%'>

###### 问题描述

日志控制要求在Banner、课程分类列表、公开课列表、猜你喜欢、精选课程等模块出现在屏幕时发送一条日志，滑出屏幕再滑入也应该重新发送，其中Banner、课程分类列表和公开课列表是可以横向滑动的，滑动时要发送展示部分的日志，每个部分来回滚动也仅发送一次。

因为要获取各个部分的offsetTop和height，所以目前使用的方案是使用EventEmitter，在各组件onLayout的时候将组件的offsetTop和height发送给日志管理(LogControl)组件，为了方便使用，组件使用了[单例模式](http://www.cnblogs.com/TomXu/archive/2012/02/20/2352817.html),并继承了EventEmitter，在需要发送日志的组件下引入LogControl实例来传递信息，在Home组件的滚动事件下监听各组件的状态，如进入屏幕则发送日志。

###### 相关代码

```javascript
import Events from 'event-emitter'
export default function LogControl() {
    // 判断是否存在实例
    if (typeof LogControl.instance === 'object') {
        return LogControl.instance;
    }
    
    this._listObject = {};
    this._offsetTopList = {};

    // 缓存
    LogControl.instance = this;
}
LogControl.prototype = new Events()
LogControl.prototype.constructor = LogControl;
```
###### 遇到问题及解决方案

*  安卓在组件滑动时会重复调用onLayout事件，导致日志重复发送。<br>
解决方案：在组件内增加一个属性判断是否触发过onLayout事件，是则不再调用。
*  安卓在初次进入首页加载完成后不会自动滚动，而IOS下会有滚动效果，导致安卓下只有在滑动了屏幕之后才会发送日志。<br>
解决方案一：安卓在加载完毕后手动调用scrollTo方法去触发一次onScroll事件。(未采用)<br>
解决方案二：在Home组件componentDidUpdate中调用一次LogControl中的方法去发送日志。

#### 二、搜索框提取

###### 问题描述

因为垂直页改版后需要使用到搜索框，故将原本只是用于首页的搜索框组件提取为公用组件，将相关样式改为了可配置的形式，由父组件通过参数传入，因为首页和垂直页的搜索页一致，所以保留了其样式，日后如有需要再进行修改。

###### 相关代码

```javascript
      <Search
        isYoudaoCourseApp={this.state.isYoudaoCourseApp}
        // setScrollViewEnableFunc={this.setScrollViewEnableFunc}
        hasSearchBg={this.state.hasSearchBg}
        setBarStyleFunc={this.setBarStyleFunc}
        fadeAnim={this.state.fadeAnim}

        searchContainerStyle={searchStyle.containerSearch}
        searchOuterHideBarStyle={searchStyle.outerHideBar}
        searchOuterStyle={searchStyle.outer}
        searchOuterBgStyle={searchStyle.outerBg}
        searchPlaceHolderStyle={searchStyle.searchPlaceHolder}
        searchInputContainerStyle={searchStyle.searchInputCon}
        searchInputStyle={[searchStyle.searchInput, {backgroundColor: this.state.hasSearchBg ? 'rgba(233,233,233,.8)' : 'rgba(255,255,255,.8)'}]}
        searchIconStyle={searchStyle.icSearch}
      />
```

#### 三、导航栏

###### 布局

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/vertical3.jpg' width='31%'>
<span>旧版</span>
</div>

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/vertical1.jpg' width='31%'>
<span>新版</span>
</div>

###### 问题描述

不同于旧版，新版导航去掉了原来的滚动条，改为了垂直标题+搜索框的形式。

###### 遇到问题及解决方案

*  React native的元素堆叠顺序无法通过zIndex，所以如果将导航组件写在最前面的话，搜索页会被下面的ScrollView遮盖。<br>
解决方案：将搜索框改为绝对定位并置于文档最后。
*  导航标题字数不一，如果搜索框宽度固定，可能会与标题重叠。<br>
解决方案：在原有导航位置放置一个仅有背景色和高度的View组件，将标题和搜索框作为一个整体放置在文档最下面，然后通过绝对定位覆盖在View组件上层，此时搜索框就可以设置为自适应宽度了。

```javascript
      <View style={[styles.container, Platform.OS === 'android' && !isTeacher ? {marginTop: tag.get('hideStatusBar') ? statusBarHeight : 0} : {marginTop: 0}]}>
        {isTeacher ? null : <View style={styles.headNav}/>} // 这个<View>仅用于占位
        <ScrollView>
            ...
        </ScrollView>
        {this._renderFixedNav(tag, this.state.isYoudaoCourseApp)} // 真正的导航栏
      </View>
```

#### 四、Banner

###### 问题描述

如上图，新版Banner每个图片并不占据整个屏宽，两边露出上下两张图片的一小部分，以做WEB的滑动组件的经验来说，要实现这样的功能，无非也就是通过绝对定位设置滚动栏，滚动时通过改变left或者translate来改变位置，如下图：

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/prototype.png' width='80%'>
</div>

类推到这里，想要实现新版的效果，只需要将外层容器宽度设置成对应的数值，在设置overflow:visible即可，如下图：

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/prototype2.png' width='80%'>
</div>

在IOS端，一切正如我所料，相当之顺利，但是拿起安卓机一看，好像不太对劲，并没有出现预期的效果，Google一番得知，安卓不支持overflow属性！？

由于原本使用的是第三方的[react-native-swiper组件](https://github.com/leecade/react-native-swiper)，出现这种情况赶紧翻看一下源码，看看能不能找到什么解决方案，然后发现在IOS端Swiper使用的是ScrollView，而在Android端使用的是ViewPagerAndroid，找了个安卓的朋友问了问，在原生安卓上使用ViewPager是可以实现这样的效果的([ViewPager实现一个页面多个Item的显示](http://m.blog.csdn.net/hb8676086/article/details/50628429))，然而，ViewPagerAndroid并没有提供诸如clipChildren、layerType的属性，只能寻求别的方案了。

后来决定用Animate自己写一个滑动组件出来，写了个小demo，发现十分卡顿，可能姿势不对吧。

奋斗几天无果，后来在网上看到[react-native-viewpager组件](https://github.com/race604/react-native-viewpager)，无奈之下下载来看看源码，居然也是用Animate写的，感觉有戏！为了实现设计稿的效果，改了一下源码并拷贝出来作为一个自己的组件来使用。

用这个组件虽然实现了想要的效果，但是性能相较于ViewPagerAndroid确实要低一些，滑动过程中会有些许卡顿，为了不影响IOS端，IOS端还是保留了原来的写法，仅在Android端使用。

###### 相关代码
ViewPager组件源码修改
```javascript
    var offset = this.props.offset; // 加入offset属性来设置偏移
    // this.childIndex = hasLeft ? 1 : 0;
    // this.state.scrollValue.setValue(this.childIndex);
    var translateX = this.state.scrollValue.interpolate({
      inputRange: [0, 1], outputRange: [offset, -viewWidth + offset] // 修改了滑动范围
    });
```
ViewPager组件调用
```javascript
                <ViewPager
                  dataSource={ds}
                  renderPageIndicator={false}
                  isLoop={ds.pageIdentities.length > 1}
                  autoPlay={true}
                  offset={calculatePixel(16)}
                  childWidth={calculatePixel(328)} // 定义每个子元素的实际宽度（加入了边距）
                  renderPage={this._renderBannerItem.bind(this)}
                />
```

#### 五、其他部分

###### 布局

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/vertical4.jpg' width='31%'>
</div>

###### 问题描述

如上图布局，更新还包括了课程分类的更新、加入了图片标题、课程入口的更新。

这一部分比较简单，似乎没啥好说的。

### 优化方案

#### 一、减少过度绘制

在安卓机的开发者选项中可以开启“调试GPU过度绘制”，如下图：

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/android1.png' width='33%'>
</div>

关于安卓过度绘制的详情可以在[这里](http://blog.csdn.net/moyameizan/article/details/47807327)查看，简单来说就是界面元素的多重层叠，假设每层元素都有背景，那么对于用户来说，只有最上层的背景才是可以看到的，其它的背景虽然绘制了，但是却没有起到效果，就是过度绘制了。

安卓GPU过度绘制的颜色信息大致如下：

>  *  蓝色1x过度绘制
>  *  绿色2x过度绘制
>  *  淡红色3x过度绘制
>  *  红色超过4x过度绘制

颜色越浅表示过度绘制程度越低，原色表示没有过度绘制。

现在来看看自己的APP会呈现出什么效果：

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/android2.png' width='33%'>
<span>首页</span>
</div>

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/android3.png' width='33%'>
<span>垂直页</span>
</div>

首页和垂直页差距似乎有点大，这里看到垂直页基本满屏大红，导致这个问题的原因不是垂直页充满了大量的背景，而是路由切换并没有把首页隐藏，垂直页相当于一整个元素覆盖在首页上方，所以看到的满屏大红是首页绘制加上垂直页绘制的效果，所以我们似乎找到了一个可以优化的地方：<em>如何在路由切换的时候将首页隐藏或者像原生APP那样切换到一个新的界面？</em>

#### 二、bundle拆包

一般来说，一个简单的RN应用，打包之后的bundle会有500+KB是属于RN的依赖，与业务无关，而我们的APP将安卓打包之后生成的bundle有900+KB，其中绝大部分应该也是来自各种依赖文件，如果能将依赖和业务文件拆分开来，生成一个common.bundle、一个或多个business.bundle，那么我们可以在一定程度上改善用户体验。

>  *  减少初始时间（提前运行基础代码）
>  *  部分更新
>  *  在多个bundle之间共享公共模块

<div align='center'>
<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/bundle.png' width='80%'>
</div>

上图引自[issue/5399](https://github.com/facebook/react-native/issues/5399)，在用户进入应用之前，我们就可以加载并运行common.bundle，并在用户进入应用之后加载指定的业务文件，而不必一次性把所有东西都加载进行，以提升性能。

目前可参考的拆包方案有

*  [携程是如何做React Native优化的](https://zhuanlan.zhihu.com/p/23715716)，[moles-packer](https://github.com/ctripcorp/moles-packer)(携程似乎已经放弃这个方案，改为以unbundle为基础的拆包方案)
*  [【React Native】一个简单的拆分Bundle&资源做法](https://blog.desmondyao.com/rn-split/)
*  [React Native Bundle Split](http://coofee.github.io/post/react-native-bundle-split/)
*  [react-native-split](https://github.com/desmond1121/react-native-split)

### 问题总览

*  安卓RN不支持overflow属性
*  安卓ScrollView等组件在滑动的时候会触发自己和其他组件的onLayout事件
*  安卓line-height属性不支持小数
*  安卓在背景色过度设置的时候会严重影响性能
