# You awesome recipe for video enabled webviews

This is your ultimate recipe with minimal changes into supporting full screen webview videos, that could be used anywhere in your application.

# Issues this guide will help you fix

* Enable full screen toggle in an Activity/Fragment
* Enable full screen toggle in Recyclerview/Nested Recyclerview
* Structure your xml views accordingly to avoid any glitches during toggling

# Cause of the problem

According to android docs, if a simple quesiton asked, what does a webview do ? The answer would be,
```
All that WebView does, by default, is show a web page.
```

Limited capabilities, very little control over the way webview work or look, requires a lot of changes in the FE side if you would like to perform a specific action, and would probably need to implement an interface to deal with some of the UI components in your view.

One of the major challenges is video handling in webviews, videos are just displayed as videos, that being said, one very big issue is full screen handling which is not enabled by default, one reason for this how the android structures UI into XMLs, and inflates those XMLs into a parent activity that holds whatever views inside, you have to tell android how to toggle and how to inflate the two modes **Full screen** and **Normal screen**.

# Breaking down the solution

That being said, enough chitchat, let us dig into the actual steps taken for this fix:

 1. Locate and find the normal layout for the webview in your current view that is inflated onto your running instance.

 2. Locate and find an empty layout that you need your webview to be inflated into during **Full screen** mode.

 3. Implement a custom **WebChromeClient**, a client is the engine of a webview, **WebViewClient** lacks a lot of features though, only to be found in its chrome sibling, includes some powerful tools like javascript invoking, ability to override favicon and of course **ability to handle custom views on full screen toggle**.

 4. (Bonus tip) You have a recyclerview ? No worries, consider your parent still as your holding activity, implement an interface that deals between the viewholder and the activity to toggle full screen.

 5. Put the pieces together, and watch the magic working seamlessly.

# Code snippets
### 1. Locating normal layout (Layout that contains normal view)
<img align="right" width="200" src="https://i.imgur.com/KbLpxEI.jpg">

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:layout_marginBottom="@dimen/dimen_5_dp">
    
    ... LAYOUT TAGS GOES HERE

    <WebView
        android:id="@+id/chat_item_video_player"
        android:layout_width="match_parent"
        android:layout_height="@dimen/dimen_120_dp"
        android:layout_marginStart="@dimen/dimen_10_dp"
        android:layout_marginEnd="@dimen/dimen_10_dp"
        android:layout_marginTop="@dimen/dimen_5_dp"
        android:background="@drawable/shape_rect_rounded_gray_background" />
        
    ... OTHER LAYOUT TAGS GOES HERE
    
</LinearLayout>
```

### 2. Create an empty view, matching layout params with parent layout.
<img align="right" width="200" src="https://i.imgur.com/yAi7xpQ.jpg">

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/cl_parent"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/white"
    tools:context=".ui.fragments.ChatActivity"> <-- THIS IS YOUR PARENT/HOLDING LAYOUT

    ... LAYOUT TAGS GOES HERE

    <FrameLayout
        android:id="@+id/fl_full_video_player"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/black"
        android:visibility="gone" />
        
    ... OTHER LAYOUT TAGS GOES HERE

</android.support.constraint.ConstraintLayout>
```

### 3. Create an interface that communicates with the holding activity
```kotlin
private interface OnFullScreenToggleListener {
    fun onShowFullScreen(view: View)
    fun onHideFullScreen(view: View)
}
```

### 4. Create a custom web chrome client that overrides the custom view handling
Hint: Create a private custom view reference to pass it on hide custom view callback, and initialize the mFullScreenListener upon creating the child view
```kotlin
private class CustomWebClient : WebChromeClient() {
    override fun onShowCustomView(view: View, callback: WebChromeClient.CustomViewCallback) {
        mFullScreenListener.onShowFullScreen(view)
        mCustomView = view
    }

    override fun onHideCustomView() {
        if (mCustomView == null)
            return

        mFullScreenListener.onHideFullScreen(mCustomView)
        mCustomView = null
    }
}
```

### 5. Initialize the webview with the custom web client
```kotlin
chat_item_video_player.setWebChromeClient(CustomWebClient())
```

### 6. Implement the interface in your target activity/fragment
```kotlin
override fun onShowFullScreen(view: View) {
    fl_full_video_player.addView(view)
    fl_full_video_player.setVisibility(View.VISIBLE)
}

override fun onHideFullScreen(view: View) {
    fl_full_video_player.removeView(view)
    fl_full_video_player.setVisibility(View.GONE)
}
```

### 7. Build, run and enjoy !
<p align="center">
  <img src="https://i.imgur.com/8GvesT0.gif" />
</p>
