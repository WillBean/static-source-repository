## React Native实战总结

入职有道数月，主要参与了精品课垂直页的改版，期间遇到了不少坑，虽然还未正式上线，但是也值得总结一波，故写下此文。

### 需求介绍

*  首页部分：加入日志控制工具以收集、统计用户操作；改写搜索框写法，将搜索框提取成独立的共用组件。
*  垂直页部分：此次改版集中在垂直页，新版垂直页将导航部分改版加入搜索框；重新设计了Banner部分；课程入口信息增多样式改变；将原本的两列布局改成新版的一列布局，还加入了原本没有的（图片）小标题。

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

#### 五、课程入口
#### 六、小标题

### 优化
