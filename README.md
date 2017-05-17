# 微课 O 版 MVP 模式设计方案

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

![What's MVP?](https://raw.githubusercontent.com/NeoFocus/WeikeOVersionMVP/master/Res/mvp.png)

通过上图我们可以清楚的看到，使用 MVP 模式能够有效的对 View 和 Model 解耦。Presenter 负责与 Model 进行交互。如果需要 UI 更新，通过 View 提供的接口进行操作。View 也持有 Presenter 的引用，来执行用户操作请求。

### 2. Why MVP ?

MVP 在 Android 上的使用其实已经有挺长一段时间了，似乎有点 “过时” 了，那为什么现在还要用 MVP。今天我想要讨论它的主要原因有如下几点：
1. MVP 并未过时，值得学习
2. MVP 易于理解，实现较易，同时也足够优秀
3. 鉴于之前的项目并未采用框架而导致 Activity/Fragment 内代码混乱不堪，MVP 是一个能够较快实现代码重构的框架，对我们是合适的

目前关于 MVP 的资料杂乱繁多、写法各异，本文参考 Google 官方提供的 MVP 架构提供一个范本

### 3. 我是怎么做的

我们以 Vip 页面为例，先看一下项目结构图：

![Structure](https://raw.githubusercontent.com/NeoFocus/WeikeOVersionMVP/master/Res/structure.png)

项目是按照功能模块组织的，Vip 包外定义了所有模块 Presenter 和 View 的基础公共接口，内容非常简单：
```java
/**
  * start 方法表示 Presenter 开始工作
  */
public interface BasePresenter {
    void start();
}
/**
  * 用泛型来接收不同的 Presenter
  */
public interface BaseView<T> {
    void setPresenter(T presenter);
}
```
setPresenter 设置 View 对 Presenter 的引用。start 是 View 显式通知 Presenter 开始工作的接口。

**MVP 中的对应关系：**

Model：VipModel

View：VipFragment

Presenter：VipPresenter

其中 VipFragment 和 VipPresenter 是通过契约类来进行连接的，也就是图中的 VipContract 。在这里 Google 引入了一个新的叫做契约类的中间部分，包括对于某一具体模块中对于 View 和 Presenter 的方法的定义。我们可以以 VipContract  为例来看一下这个契约类具体是用来干什么的：

```java
interface VipContract {
    interface View extends BaseView<Presenter> {
        void showVipContent(VipPriceInfo info);

        void turnOrderPage(PaymentInfo info);

        boolean isActive();

        void showMyVipInfo(VipMyInfo info);
    }

    interface Presenter extends BasePresenter {
        void submitOrder(String vipId);
    }
}
```

这个类的主要内容是根据当前的模块来派生出适合于当前模块的 View 和 Presenter ，在这里将 View 和与之对应的 Presenter 绑定在一起，而且集中的显示了 View 和 Presenter 的功能。接口中的方法是根据当前模块中具体的 View 和 Presenter 来设计的。实现的时候 View 和 Presenter 直接实现这里面的各自的接口就可以了，好处是有什么方法都会在这里看到，清晰简洁一目了然。

**具体的 Model 实现**

Model 的实现与具体业务相关，这里主要是发起网络请求，并将请求结果回调给 Presenter 由 Presenter 来对数据做相应的处理，没有什么好说的，写法也很自由，这里并没有采用 Google Demo 内的写法，因为和我们业务不太相符。

```java
class VipModel {
    private static final ExecutorService sExecutor = Executors.newFixedThreadPool(2);
    private VipPriceInfo mPriceInfo;
    private VipMyInfo mMyInfo;
    private VipOrderInfo mOrderInfo;
    private VipModelListener mModel;

    void setListener(VipModelListener model) {
        mModel = model;
    }

    void doLoadMyInfo() {
        AsyncTask<Void, Void, ErrorHandler> loadMyTask
                = new AsyncTask<Void, Void, ErrorHandler>() {
            @Override
            protected ErrorHandler doInBackground(Void p) {
                ErrorHandler error = new ErrorHandler();
                mMyInfo = Parser.parseVipMyInfo(error);
                return error;
            }

            @Override
            protected void onPostExecute(ErrorHandler results) {
                super.onPostExecute(results);
                if (!results.success()) {
                    return;
                }
                mModel.onMyInfoLoaded(mMyInfo);
            }
        };
        loadMyTask.execute(sExecutor, null);
    }

    void doLoadVipInfo() {
        AsyncTask<Void, Void, ErrorHandler> loadVipTask
                = new AsyncTask<Void, Void, ErrorHandler>() {
            @Override
            protected ErrorHandler doInBackground(Void p) {
                ErrorHandler error = new ErrorHandler();
                mPriceInfo = Parser.parseVipPriceInfo(error);
                return error;
            }

            @Override
            protected void onPostExecute(ErrorHandler results) {
                super.onPostExecute(results);
                if (!results.success()) {
                    return;
                }
                mModel.onVipInfoLoaded(mPriceInfo);
            }
        };
        loadVipTask.execute(sExecutor, null);
    }

    void doSubmitOrder(String vipId) {
        AsyncTask<Void, Void, ErrorHandler> submitTask
                = new AsyncTask<Void, Void, ErrorHandler>() {
            @Override
            protected ErrorHandler doInBackground(Void p) {
                ErrorHandler error = new ErrorHandler();
                info = Parser.parseVipOrderInfo(error, vipId);
                return error;
            }

            @Override
            protected void onPostExecute(ErrorHandler results) {
                super.onPostExecute(results);
                if (!results.success()) {
                    return;
                }
                mModel.onOrderSubmitted(info);
            }
        };
        submitTask.execute(sExecutor, null);
    }

    interface VipModelListener {
        void onMyInfoLoaded(VipMyInfo info);

        void onVipInfoLoaded(VipPriceInfo info);

        void onOrderSubmitted(PaymentInfo info);
    }
}
```

**具体的 View 和 Presenter 的实现：**

```java
public class VipFragment extends AbsBaseFragment implements VipContract.View {

    public static VipFragment newInstance() {
        return new VipFragment();
    }
  
    @Override
    public void setPresenter(VipContract.Presenter presenter) {
        mPresenter = checkNotNull(presenter);
    }
  
    @Override
    public void onResume() {
        super.onResume();
        mPresenter.start();
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
      
        initView();
    }

	privat void initView(){
        //.......
        mShowOrderButton.setOnClickListener(v -> { mPresenter.submitOrder(mVipId);});
        //.......
	}

    @Override
    public void showVipContent(VipPriceInfo info) {
        mPriceInfo = info;
        mListAdapter.setData(info);
        mTvPayAmount.setText(info.getData().get(mCurrentSelectPosition).getPrice());
        mVipId = info.getData().get(mCurrentSelectPosition).getId();
    }

    @Override
    public void turnOrderPage(PaymentInfo info) {
        Intent intent = new Intent(getActivity(), OderActivity.class);
        getActivity().startActivityForResult(intent, 0);
    }

    @Override
    public boolean isActive() {
        return isAdded();
    }

    @Override
    public void showMyVipInfo(VipMyInfo info) {
        setMyInfo(info);
    }
}
```

上边这个 VipFragment 就是 View 层的实现，基本上体现了 “只是进行数据的显示，并不进行业务逻辑的处理” 这种思想。

在 onResume 方法里调用了 Presenter 的 start 方法，用来告知 Presenter View 已经初始化完成当前处于活动状态。当按钮被点击后也涉及到了业务逻辑，所以业务逻辑就要交给了 Presenter 来处理。

接下来看一下Presenter层的实现示例：

```java
class VipPresenter implements VipContract.Presenter, VipModel.VipModelListener {
    private VipContract.View mVipView;
    private VipModel mVipModel;

    VipPresenter(VipContract.View view, VipModel model) {
        mVipView = view;
        mVipModel = model;
        mVipView.setPresenter(this);
        mVipModel.setListener(this);
    }

    @Override
    public void start() {
        mVipModel.doLoadMyInfo();
        mVipModel.doLoadVipInfo();
    }

    @Override
    public void submitOrder(String vipId) {
        mVipModel.doSubmitOrder(vipId);
    }

    @Override
    public void onMyInfoLoaded(VipMyInfo info) {
        if (mVipView.isActive()) {
            mVipView.showMyVipInfo(info);
        }
    }

    @Override
    public void onVipInfoLoaded(VipPriceInfo info) {
        if (mVipView.isActive()) {
            mVipView.showVipContent(info);
        }
    }

    @Override
    public void onOrderSubmitted(PaymentInfo info) {
        if (mVipView.isActive()) {
            mVipView.turnOrderPage(info);
        }
    }
}
```

Presenter 中主要是实现 View 和 Model 的交互，并且将两者隔离开来，实现了接口隔离。VipPresenter 实现 VipContract.Presenter 契约类接口，在构造里面接受 View 的对象，并且调用了View 的 setPresenter 方法，把自己的引用传了进去，这就是上面基类定义的方法，这样 Presenter 就有了 View 的引用  View 也有了Presenter 的引用。而实现 VipModel.VipModelListener 接口是为了响应 Model 网络请求完成后的回调。而且，将业务逻辑放到 Presenter 中去，可以方便去驱动避免 Activity 退出而后台线程仍然引用着 Activity，致使资源无法被系统回收从而引起内存泄露。

还可以看到上面实现了start 方法并调用了 doLoadMyInfo 和 doLoadVipInfo 方法，这两个方法其实是用来加载 View 初始化完成后所需要的数据的，在加载完成后讲数据交给 View 来使用。

**那 Activity 哪去了？**

最后要讲就是 VipActivity ，我们知道 Activity 和 Fragment 都可以做 View 层，那为什么不用 Activity 做 View 层呢，主要是因为 Activity 是一个过于万能的类，而且操作必须切只能在 Activity 内来完成，如果用它来做 View 层，那么 View 层就不可避免的会出现许多和 View 无关的代码，最后和我们设计的初衷背离。

在这里我们主要是用它做一个承载的作用，类似于一个容器：

```java
public class VipActivity extends AbsToolbarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_vip);
        setTitleBarView(R.layout.action_bar_title);

        initMVP();
        // and more...
    }

    private void initMVP() {
        VipModel model = new VipModel();

        FragmentManager manager = getSupportFragmentManager();
        VipFragment vipFragment = (VipFragment) manager.findFragmentById(R.id.vip_content);

        if (vipFragment == null) {
            vipFragment = VipFragment.newInstance();
            ActivityUtils.addFragmentToActivity(manager, vipFragment, R.id.vip_content);
        }

        new VipPresenter(vipFragment, model);
    }
    // ...... 其他 Activity 代码
}
```

可以看到在 MVP 中，VipActicity 只负责创建了 VipModel（Model）、 VipFragment（View）和 VipPresenter（Presenter），并将他们装配在了一起，这就是 Activity 的作用。

### 4.  总结

总结下来，感觉就是 Activity 像是一个舞台，承载着上边所有的元素进行表演，而 Presenter 这是提线木偶中提线的那个人，操控着任务的交互，View 则是木偶的动作表演，而 Model 这是推动故事发展的剧情。

因为剧情的发展（Model 的改变）导致提线人知道剧情发生变化（Presenter 得知 Model 发生了变化）从而操作木偶（View 发生变化）产生动作。

同样的，当木偶产生动作时（View 发生变化），Presenter 知道木偶发生了这个动作（Presenter 知道 View 发生了变化）之后下一步剧情该怎么发展（Model 怎么改变）。

这是本人参考 Google 官方给出的 MVP 模式 Demo 设计会员详情界面的 MVP 实现，虽然功能很简单代码量也很少，但是通过这个模块的设计，让我更多的学会了去思考和设计给了我很启发，而不只是单纯的实现功能。本人才疏学浅，有疏漏之处欢迎讨论。



### 链接

有兴趣的可以在  [Github](https://github.com/googlesamples/android-architecture) 上看看 Google 官方给出的 MVP 实例的源码。