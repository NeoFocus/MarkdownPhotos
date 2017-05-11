# 微课 O 版 MVP 架构设计方案
1. ## What's MVP ?
  ### 先来看一段百度上的解释
  > 如此如此这般这般
  > 嗯哼？

  ![What's MVP?](https://raw.githubusercontent.com/NeoFocus/MarkdownPhotos/master/Res/1233754-eb5b4bc4fbf757be.png)

2. ## Why MVP ?

    阿三打算发

3. ## 我是怎么做的


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
4. ## 总结