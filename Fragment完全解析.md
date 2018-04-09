ViewPager源码解析

ViewPagerAdapter、FragmentPagerAdapter与FragmentStatePagerAdapter的区别

Fragment 、 FragmentManager、FragmentTransaction

在Android中，对`Fragment`的操作都是通过`FragmentTransaction`来执行。而从`Fragment`的结果来看，`FragmentTransaction`中对`Fragment`的操作大致可以分为两类：

- 显示：`add() replace() show() attach()`

- 隐藏：`remove() hide() detach()`
  对于每一组方法，虽然最后产生的效果类似，但方法背后带来的副作用以及对Fragment的生命周期的影响都不尽相同。

`FragmentPagerAdapter`主要方法

```java
@Override
public Object instantiateItem(ViewGroup container, int position) {
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }

    final long itemId = getItemId(position);

    // Do we already have this fragment?
    String name = makeFragmentName(container.getId(), itemId);
    Fragment fragment = mFragmentManager.findFragmentByTag(name);
    if (fragment != null) {
        if (DEBUG) Log.v(TAG, "Attaching item #" + itemId + ": f=" + fragment);
        mCurTransaction.attach(fragment);
    } else {
        fragment = getItem(position);
        if (DEBUG) Log.v(TAG, "Adding item #" + itemId + ": f=" + fragment);
        mCurTransaction.add(container.getId(), fragment,
                makeFragmentName(container.getId(), itemId));
    }
    if (fragment != mCurrentPrimaryItem) {
        fragment.setMenuVisibility(false);
        fragment.setUserVisibleHint(false);
    }

    return fragment;
}
```


```java
  @Override
public void destroyItem(ViewGroup container, int position, Object object) {
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }
    if (DEBUG) Log.v(TAG, "Detaching item #" + getItemId(position) + ": f=" + object
            + " v=" + ((Fragment)object).getView());
    //执行了detach
    mCurTransaction.detach((Fragment)object);
}
```
- ` mCurTransaction.attach(fragment);`

  时`Fragment`的生命周期执行顺序是从`onCreateView`方法开始


- ` mCurTransaction.detach((Fragment)object);`

  时`Fragment`的生命周期执行了`onDestoryView()`,并未继续执行`onDestory()`和`onDetach()`;

```java
//设置主item 即显示的item
@Override
public void setPrimaryItem(ViewGroup container, int position, Object object) {
    Fragment fragment = (Fragment)object;
    if (fragment != mCurrentPrimaryItem) {
        if (mCurrentPrimaryItem != null) {
            mCurrentPrimaryItem.setMenuVisibility(false);
            mCurrentPrimaryItem.setUserVisibleHint(false);
        }
        if (fragment != null) {
            fragment.setMenuVisibility(true);
            fragment.setUserVisibleHint(true);
        }
        mCurrentPrimaryItem = fragment;
    }
}
```


```java
@Override
public void finishUpdate(ViewGroup container) {
    if (mCurTransaction != null) {
        mCurTransaction.commitNowAllowingStateLoss();//关键代码不明白意思？
        mCurTransaction = null;
    }
}
```
`FragmentStatePagerAdapter`主要方法

```java
@Override
public Object instantiateItem(ViewGroup container, int position) {
    // If we already have this item instantiated, there is nothing
    // to do.  This can happen when we are restoring the entire pager
    // from its saved state, where the fragment manager has already
    // taken care of restoring the fragments we previously had instantiated.
    if (mFragments.size() > position) {
        Fragment f = mFragments.get(position);
        if (f != null) {
            return f;
        }
    }

    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }

    Fragment fragment = getItem(position);
    if (DEBUG) Log.v(TAG, "Adding item #" + position + ": f=" + fragment);
    if (mSavedState.size() > position) {
        Fragment.SavedState fss = mSavedState.get(position);
        if (fss != null) {
            fragment.setInitialSavedState(fss);
        }
    }
    while (mFragments.size() <= position) {
        mFragments.add(null);
    }
    fragment.setMenuVisibility(false);
    fragment.setUserVisibleHint(false);
    mFragments.set(position, fragment);
    mCurTransaction.add(container.getId(), fragment);

    return fragment;
}
```

```java
@Override
public void destroyItem(ViewGroup container, int position, Object object) {
    Fragment fragment = (Fragment) object;

    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }
    if (DEBUG) Log.v(TAG, "Removing item #" + position + ": f=" + object
            + " v=" + ((Fragment)object).getView());
    while (mSavedState.size() <= position) {
        mSavedState.add(null);
    }
    mSavedState.set(position, fragment.isAdded()
            ? mFragmentManager.saveFragmentInstanceState(fragment) : null);
    mFragments.set(position, null);

    mCurTransaction.remove(fragment);
}
```