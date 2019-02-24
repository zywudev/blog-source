---
title: Android ViewPager 与 Fragment 懒加载
date: 2018-04-23 20:59:05
tags: Android
toc: true
---

ViewPager + 多 Fragment 的模式很常用，但是 ViewPager 存在预加载的问题，如果多个 Fragment 都存在大量的网络请求或读写情况，就影响了 APP 性能和体验。在网上找到了一个比较好的[解决方法](https://www.jianshu.com/p/c5d29a0c3f4c#)，方法就是保留 ViewPager 的预加载，在 Fragment 被选中时再加载数据，记录一下。

主要的代码如下：

```java
/**
 * Created by wuzy on 2018/4/23.
 **/
public abstract class BasePagerFragment extends BaseFragment {

    private static final String TAG = "BasePagerFragment";

    protected boolean isViewInitiated;    
    protected boolean isVisibleToUser;
    protected boolean isDataInitiated;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        isViewInitiated = true;
        prepareLoadData();
        Log.e(TAG, "onActivityCreated: ");
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        this.isVisibleToUser = isVisibleToUser;
        prepareLoadData();
        Log.e(TAG, "setUserVisibleHint: " );
    }

    public abstract void loadData();

    public boolean prepareLoadData() {
        return prepareLoadData(false);
    }

    public boolean prepareLoadData(boolean forceUpdate) {
        if (isVisibleToUser && isViewInitiated && (!isDataInitiated || forceUpdate)) {
            loadData();
            isDataInitiated = true;
            return true;
        }
        return false;
    }
}
```

这里简单解释下，`setUserVisibleHint` 方法可被用来判断 Fragment UI 是否可见。注意的是，如果项目中没有使用 ViewPager 的话，这个方法压根不调用。
这里，在 UI 可见的时候的时候 调用 `prepareLoadData` 方法，在 `prepareLoadData` 方法中做判断：如果 UI 可见，并且 Fragment 已经初始化完毕，数据未加载或者需要强制加载数据的情况下，进行数据加载。

在某些情况下可能想禁掉 ViewPager 的滑动，可以自定义 ViewPager，提供设置滑动的方法。

```java
/**
 * Created by wuzy on 2018/4/23.
 **/
public class CustomViewPager extends ViewPager {

    private boolean isPageChangeEnabled = true;

    public CustomViewPager(Context context) {
        super(context);
    }

    public CustomViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return this.isPageChangeEnabled && super.onTouchEvent(event);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        return this.isPageChangeEnabled && super.onInterceptTouchEvent(event);
    }

    /**
     * 设置是否可以滑动
     * @param b
     */
    public void setPageChangeEnabled(boolean b) {
        this.isPageChangeEnabled = b;
    }
}
```

具体 Demo 地址 ：[ViewPagerDemo](https://github.com/zywudev/ViewPagerDemo)。





