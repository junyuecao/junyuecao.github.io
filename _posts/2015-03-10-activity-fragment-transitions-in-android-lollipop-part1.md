---
layout: post
title: Activity&Fragment Transition(过渡动画)入门 part 1
description: ""
category: Java
tags: ["Java", "Android", "Animation"]
---

{% include JB/setup %}

> 本文翻译自: [http://www.androiddesignpatterns.com/2014/12/activity-fragment-transitions-in-android-lollipop-part1.html](http://www.androiddesignpatterns.com/2014/12/activity-fragment-transitions-in-android-lollipop-part1.html)

> 为了更好地理解, 部分术语不进行翻译:Transition , Scene, Activity, Fragment ...

本文将会对Transition进行简短的介绍, 然后介绍一下Android5.0 Lollipop里面引入的全新的[Activity & Fragment 的Transition API](https://developer.android.com/training/material/animations.html#Transitions). 整个系列主要介绍以下几个主题:

 - Part 1: Activity & Fragment Transition入门
 - Part 2: 深入内容Transition(Content Transitions)
 - Part 3a: 深入共享元素Transition(Shared Element Transitions)
 - Part 3b: 深入延迟共享元素Transition(Postponed Shared Element Transitions)
 - Part 3c: 实现共享元素回调(Shared Element Callbacks) (coming soon!)
 - Part 4: Activity & Fragment Transition例子 (coming soon!)


首先我们来回答一下这个问题: 什么是Transition?

### 什么是`Transition`呢?

棒棒糖中的 Activity & Fragment 的Transition是建立在一个相对比较新的特性上的,这个特性我们称之为`Transition`.它是在KitKat中引入的, Transition 框架提供了便捷的API来实现应用程序在不同的UI状态之间的动画. 这个框架主要围绕两个概念来设计: Scenes(场景)和Transitions. Scnene定义了一个应用程序UI的特定的状态, Transition定义了两个场景之间的动画.

当一个Scene变化的时候, Transition主要有两个职责:

 1. 获取每个View的状态, 包括Scene的开头和结尾处的
 2. 创建一个基于这两者差异的Animator,这个Animator实现从一个Scene切换到另一个Scene的动画.


例如, 我们假设当用户点击屏幕时,一个Activity将会渐入和渐出他的全部View. 我们通过Android的Transition框架可以只写几行代码就实现这个效果.代码如下:

{% highlight java %}
public class ExampleActivity extends Activity implements View.OnClickListener {
    private ViewGroup mRootView;
    private View mRedBox, mGreenBox, mBlueBox, mBlackBox;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mRootView = (ViewGroup) findViewById(R.id.layout_root_view);
        mRootView.setOnClickListener(this);

        mRedBox = findViewById(R.id.red_box);
        mGreenBox = findViewById(R.id.green_box);
        mBlueBox = findViewById(R.id.blue_box);
        mBlackBox = findViewById(R.id.black_box);
    }

    @Override
    public void onClick(View v) {
        TransitionManager.beginDelayedTransition(mRootView, new Fade());
        toggleVisibility(mRedBox, mGreenBox, mBlueBox, mBlackBox);
    }

    private static void toggleVisibility(View... views) {
        for (View view : views) {
            boolean isVisible = view.getVisibility() == View.VISIBLE;
            view.setVisibility(isVisible ? View.INVISIBLE : View.VISIBLE);
        }
    }
}
{% endhighlight %}

为了更好地理解这个例子, 我们假设最初的时候屏幕上每个View都是可见的, 然后我们一步一步来分析这个过程:

Video 1.1 - Running the example transition above using a Fade, Slide, and Explode. Click to play.

A click is detected and the developer calls beginDelayedTransition(), passing the scene root and a Fade transition as the arguments. The framework immediately calls the transition's captureStartValues() method for each view in the scene and the transition records each view's visibility.
When the call returns, the developer sets each view in the scene to INVISIBLE.
On the next display frame, the framework calls the transition's captureEndValues() method for each view in the scene and the transition records each view's (recently updated) visibility.
The framework calls the transition's createAnimator() method. The transition analyzes the start and end values of each view and notices a difference: the views are VISIBLE in the start scene but INVISIBLE in the end scene. The Fade transition uses this information to create and return an AnimatorSet that will fade each view's alpha property to 0f.
The framework runs the returned Animator, causing all views to gradually fade out of the screen.
This simple example highlights two main advantages that the transition framework has to offer. First, Transitions abstract the idea of Animators from the developer. As a result, Transitions can significantly reduce the amount of code you write in your activities and fragments: all the developer must do is set the views' start and end values and the Transition will automatically construct an animation based on the differences. Second, animations between scenes can be easily changed by using different Transition objects. Video 1.1, for example, illustrates the dramatically different effects we can achieve by replacing the Fade transition with a Slide or Explode. As we will see moving forward, these advantages will allow us to build complex Activity and Fragment transition animations with a relatively small amount of code. In the next few sections, we will see for ourselves how this can be done using Lollipop's new Activity and Fragment transition APIs.

Activity & Fragment Transitions in Android Lollipop

As of Android 5.0, Transitions can now be used to perform elaborate animations when switching between different Activitys or Fragments. Although Activity and Fragment animations could already be specified in previous platform versions using the Activity#overridePendingTransition() and FragmentTransaction#setCustomAnimation() methods, they were limited in that they could only animate the entire Activity/Fragment container as a whole. The new Lollipop APIs take this a step further, making it possible to animate individual views as they enter or exit their containers and even allowing us to animate shared views from one Activity/Fragment container to the other.

Let's begin by discussing the terminology that will be used in this series of posts. Note that although the terminology below is defined in terms of Activity transitions, the exact same terminology will be used for Fragment transitions as well:

Let A and B be activities and assume activity A starts activity B. We refer to A as the "calling Activity" (the activity that "calls" startActivity()) and B as the "called Activity".

The Activity transition APIs are built around the idea of exit, enter, return, and reenter transitions. In the context of activities A and B defined above, we can describe each as follows:

Activity A's exit transition determines how views in A are animated when A starts B.

Activity B's enter transition determines how views in B are animated when A starts B.

Activity B's return transition determines how views in B are animated when B returns to A.

Activity A's reenter transition determines how views in A are animated when B returns to A.

Video 1.2 - Content transitions and shared element transitions in action in the Google Play Newsstand app (as of v3.3). Click to play.

Lastly, the framework provides APIs for two types of Activity transitions—content transitions and shared element transitions—each of which allow us to customize the animations between Activities in unique ways:

A content transition determines how an activity's non-shared views—called transitioning views—enter or exit the activity scene.

A shared element transition determines how an activity's shared elements (also called hero views) are animated between two activities.

Video 1.2 gives a nice illustration of content transitions and shared element transitions used in the Google Play Newsstand app. Although we can't be sure without looking at the Newsstand source code, my best guess is that the following transitions are used:

The exit and reenter content transitions for activity A (the calling activity) are both null. We can tell because the non-shared views in A are not animated when the user exits and reenters the activity.2
The enter content transition for activity B (the called activity) uses a custom slide-in transition that shuffles the list items into place from the bottom of the screen.
The return content transition for activity B is a TransitionSet that plays two child transitions in parallel: a Slide(Gravity.TOP) transition targeting the views in the top half of the activity and a Slide(Gravity.BOTTOM) transition targeting the views in the bottom half of the activity. The result is that the activity appears to "break in half" when the user clicks the back button and returns to activity A.
The enter and return shared element transitions both use a ChangeImageTransform, causing the ImageView to be animated seamlessly between the two activities.
You've probably also noticed the cool circular reveal animation that plays under the shared element during the transition. We will cover how this can be done in a future blog post. For now, let's keep things simple and familiarize ourselves with the Activity and Fragment transition APIs.

Introducing the Activity Transition API

Creating a basic Activity transition is relatively easy using the new Lollipop APIs. Summarized below are the steps you must take in order to implement one in your application. In the posts that follow, we will go through much more advanced use-cases and examples, but for now the next two sections will serve as a good introduction:

Enable the new transition APIs by requesting the Window.FEATURE_ACTIVITY_TRANSITIONS Window.FEATURE_CONTENT_TRANSITIONS window features in your called and calling Activities, either programatically or in your theme's XML.
Set exit and enter content transitions for your calling and called activities respectively. Material-themed applications have their exit and enter content transitions set to null and Fade respectively by default. If the reenter or return transitions are not explicitly set, the activity's exit and enter content transitions respectively will be used in their place instead.
Set exit and enter shared element transitions for your calling and called activities respectively. Material-themed applications have their shared element exit and enter transitions set to @android:transition/move by default. If the reenter or return transitions are not explicitly set, the activity's exit and enter shared element transitions respectively will be used in their place instead.
To start an Activity transition with content transitions and shared elements, call the startActivity(Context, Bundle) method and pass the following Bundle as the second argument:

ActivityOptions.makeSceneTransitionAnimation(activity, pairs).toBundle();
where pairs is an array of Pair<View, String> objects listing the shared element views and names that you'd like to share between activities.3 Don't forget to give your shared elements unique transition names, either programatically or in XML. Otherwise, the transition will not work properly!

To programatically trigger a return transition, call finishAfterTransition() instead of finish().

By default, material-themed applications have their enter/return content transitions started a tiny bit before their exit/reenter content transitions complete, creating a small overlap that makes the overall effect more seamless and dramatic. If you wish to explicitly disable this behavior, you can do so by calling the setWindowAllowEnterTransitionOverlap() and setWindowAllowReturnTransitionOverlap() methods or by setting the corresponding attributes in your theme's XML.

Introducing the Fragment Transition API

If you are working with Fragment transitions, the API is similar with a few small differences:

Content exit, enter, reenter, and return transitions should be set by calling the corresponding methods in the Fragment class or as attributes in your Fragment's XML tag.
Shared element enter and return transitions should be set by calling the corresponding methods in the Fragment class or as attributes in your Fragment's XML.
Whereas Activity transitions are triggered by explicit calls to startActivity() and finishAfterTransition(), Fragment transitions are triggered automatically when a fragment is added, removed, attached, detached, shown, or hidden by a FragmentTransaction.
Shared elements should be specified as part of the FragmentTransaction by calling the addSharedElement(View, String) method before the transaction is committed.
Conclusion

In this post, we have only given a brief introduction to the new Activitiy and Fragment transition APIs. However, as we will see in the next few posts having a solid understanding of the basics will significantly speed up the development process in the long-run, especially when it comes to writing custom Transitions. In the posts that follow, we will cover content transitions and shared element transitions in even more depth and will obtain an even greater understanding of how Activity and Fragment transitions work under-the-hood.

As always, thanks for reading! Feel free to leave a comment if you have any questions, and don't forget to +1 and/or share this blog post if you found it helpful!

1 If you want to try the example out yourself, the XML layout code can be found here. ↩

2 It might look like the views in A are fading in/out of the screen at first, but what you are really seeing is activity B fading in/out of the screen on top of activity A. The views in activity A are not actually animating during this time. You can adjust the duration of the background fade by calling setTransitionBackgroundFadeDuration() on the called activity's Window. ↩

3 To start an Activity transition with content transitions but no shared elements, you can create the Bundle by calling ActivityOptions.makeSceneTransitionAnimation(activity).toBundle(). To disable content transitions and shared element transitions entirely, don't create a Bundle object at all—just pass null instead. 