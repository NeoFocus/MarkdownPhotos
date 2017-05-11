# 微课 O 版 MVP 架构设计方案
### 1. What's MVP ?
之前有一个 MVC 模式：Model-View-Controller
MVC模式 有两个主要的缺点：首先，View 持有 Controller 和 Model 的引用；第二，它没有把对 UI 逻辑的操作限制在单一的类里，这个职能被 Controller 和 View 或者 Model 共享。
所以后来提出了 MVP 模式来克服这些缺点。

先来看一段百度上的解释：

> MVP（Model-View-Presenter）模式：
> Model：即 javaBean 数据层，负责与网络层和数据库层的逻辑交互。
> View：UI层, 显示数据，并向 Presenter 报告用户行为。
> Presenter：从 Model 拿数据，应用到 UI 层，管理UI的状态决定要显示什么，响应用户的行为。
>
> MVP 模式的最主要优势就是耦合降低，Presenter 变为纯 Java 的代码逻辑，不再与 Android Framework 中的类如 Activity、Fragment 等关联，便于写单元测试。

![What's MVP?](https://raw.githubusercontent.com/NeoFocus/MarkdownPhotos/master/Res/1233754-eb5b4bc4fbf757be.png)

### 2. Why MVP ?

> MVP 在 Android 上的使用其实已经有挺长一段时间了，长到似乎有点 “过时” 了（目前风头正劲的是 MVVM ），那为什么现在还要讲 MVP。今天我想要讨论它的主要原因有如下几点：
> 1. MVP 并未过时，值得我们研究
> 2. MVP 便于理解，同时适合
> 3. 目前关于 MVP 的资料都不算太详尽，写法各异，本文提供一个范本

### 3. 我是怎么做的


```java
    private void initMVP() {
        FragmentManager manager = getSupportFragmentManager();

        mFragment = (LoginFragment) manager.findFragmentById(R.id.login_content);

        if (mFragment == null) {
            mFragment = LoginFragment.newInstance();
            ActivityUtils.addFragmentToActivity(manager, mFragment, R.id.login_content);
        }

        new LoginPresenter(mFragment);
    }
```
### 4.  总结