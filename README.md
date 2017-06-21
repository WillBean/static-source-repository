## React Native实战总结

入职有道数月，主要参与了精品课垂直页的改版，期间遇到了不少坑，虽然还未正式上线，但是也值得总结一波，故写下此文。

### 需求介绍

*  首页部分：加入日志控制工具以收集、统计用户操作；改写搜索框写法，将搜索框提取成独立的共用组件。
*  垂直页部分：此次改版集中在垂直页，新版垂直页将导航部分改版加入搜索框；重新设计了Banner部分；课程入口信息增多样式改变；将原本的两列布局改成新版的一列布局，还加入了原本没有的（图片）小标题。

### 实现方案及问题

#### 一、日志控制

###### 首页布局

<img src='/images/home1.jpg' align='left' width='31%'>

<img src='/images/home2.jpg' align='left' width='31%'>

<img src='/images/home3.jpg' align='left' width='31%'>

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
###### 遇到问题

#### 二、搜索框提取
#### 三、导航栏
#### 四、Banner
#### 五、课程入口
#### 六、小标题

### 优化
