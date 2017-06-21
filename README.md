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

<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/vertica3.jpg' width='31%'>

<img src='https://raw.githubusercontent.com/WillBean/react-native-summary.github.io/master/images/vertical.jpg' width='31%'>

###### 问题描述



###### 问题描述

#### 四、Banner
#### 五、课程入口
#### 六、小标题

### 优化
