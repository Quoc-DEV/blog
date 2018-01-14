---
layout: post
title: Kotlin extensions to commit Fragments safely
date: 2018-01-14 11:26:00 +0700
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes.
# Add post description (optional)
img: i-rest.jpg # Add image post (optional)
tags: [kotlin, android]
---

Since fragments appeared in Android, many developers have phê bình this component. The main reasons to not recommend fragments in android are; lifecycle phức tạp, hard to debug and easy to lose the fragment state.

Và lý do cuối cùng, how many of you have seen this exception in your implementations?

```
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
    at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1341)
    at android.support.v4.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1352)
    at android.support.v4.app.BackStackRecord.commitInternal(BackStackRecord.java:595)
    at android.support.v4.app.BackStackRecord.commit(BackStackRecord.java:574)
```
Mục tiêu của bài viết này là chỉ cho bạn cách để ngăn chặn the state loss fragment exception using kotlin extensions and android support library (version up to 26.0.0).

#### Why is this exception được xảy ra???

If you don’t know why this exception is thrown, I strongly recommend you to read the [Fragment Transactions & Activity State Loss article](http://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html) from [@alexjlockwood](https://twitter.com/alexjlockwood), in which he explains it in great detail.

#### Android Support Library

Since [version 26.0.0-beta 1](https://developer.android.com/topic/libraries/support-library/revisions.html#26-0-0-beta1) we can check in your app if fragments can be committed to allow the state loss or not.

> **FragmentManager** and **Fragment** have an `isStateSaved()` method to allow truy vấn của a transaction mà không có mất state. This is especially useful to check when handling an onClick() event before executing any transaction.

As you can see, the `isStateSaved()` method is really useful in order to avoid tình huống bất ngờ.

#### Kotlin extensions
Kotlin supports extension functions and extension properties.

So, **How to combine both?**

In the following example you can see how an activity extension can commit a fragment in a safe way using `isStateSaved()` method from the android support library:

```Kotlin
/**
 * Method to replace the fragment. The [fragment] is added to the container view with id
 * [containerViewId] and a [tag]. The operation is performed by the supportFragmentManager.
 */
fun AppCompatActivity.replaceFragmentSafely(fragment: Fragment,
                                            tag: String,
                                            allowStateLoss: Boolean = false,
                                            @IdRes containerViewId: Int,
                                            @AnimRes enterAnimation: Int = 0,
                                            @AnimRes exitAnimation: Int = 0,
                                            @AnimRes popEnterAnimation: Int = 0,
                                            @AnimRes popExitAnimation: Int = 0) {
    val ft = supportFragmentManager
            .beginTransaction()
            .setCustomAnimations(enterAnimation, exitAnimation, popEnterAnimation, popExitAnimation)
            .replace(containerViewId, fragment, tag)
    if (!supportFragmentManager.isStateSaved) {
        ft.commit()
    } else if (allowStateLoss) {
        ft.commitAllowingStateLoss()
    }
}
```

Then, it can be invoked directly from any activity that extends from AppCompatActivity:

```Kotlin
    private fun replaceFragment() {
        replaceFragmentSafely(
                fragment = Fragment2(),
                tag = TAG_FRAGMENT_TWO,
                containerViewId = R.id.content,
                allowStateLoss = true
        )
    }
```
Thank full and cover by [article](https://proandroiddev.com/kotlin-extensions-to-commit-fragments-safely-de06218a1f4) 
