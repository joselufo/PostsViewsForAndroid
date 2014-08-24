You never know what can you learn (and find) reading the source code of Android!

## Introduction

Recently, at work, I've been reading a lot of the most important parts of the Android source code that we,
as developers, tend to use everyday when we are building an app.

My personal recommendation is very clear: **I can't recommend enough** how important is, in order to improve your skills as a programmer, to take your time to dig, read and understand how basic classes like `View`, `ViewGroup`, `LayoutInflater`, `Context`, `Window`, `WindowManger` (and many more)… really work. I consider this act as the next step that every programmer (beginner or not) should do when has acquired some basic familiarity with the framework.

But… there is one little and insignificant caveat about what I said before: **It's not going to be an easy task!**

Why?

{<1>}![It's over 9000 lines of code!](http://i.imgur.com/ZRSsXSw.png)

So… it's better to take your time because there's a lot of things to see and learn (I'll write a post about this class, promised)!

## Do you remember the HTML &lt;blink&gt; tag?

If you remember the good ol' days of the Web, [that annoying tag was used everywhere](http://en.wikipedia.org/wiki/Blink_element) and its only purpose in life was to produce eyestrain and headaches!

Well, Android has its own implementation too! Let me introduce you **`BlinkLayout`**!

{<2>}![BlinkLayout in action!](http://i.imgur.com/JDAYZqV.gif)

Pretty scary huh?

Let's find it in the Android source code in order to analyse it. It should be located in the package `android.widget`…

{<3>}![Finding BlinkLayout in android.widget package](http://i.imgur.com/s7Fn6ta.png)

What the heck? It's not there! How is this possible? Because it's located inside of [`LayoutInflater`](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/4.4.2_r1/android/view/LayoutInflater.java?av=f#883) as a `private static class`… ¿?

Yep, you read right!

### How can I use it?

If you want to use this custom view (in the case that you want to produce desperation in your users) you can use the `<blink>` tag, e.g:

```xml
<blink
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/whatever"
            android:layout_gravity="center"/>

</blink>
````

And voilá! All your views will blink at a rate of 500ms (you can not change this value, it's hardcoded).

There are some known limitations with the current implementation:

1. It can only be instantiated only from an `XML`layout. There is no option to do it programmatically (remember, it's a private class so you [don't have access to it without resorting to black magic](http://docs.oracle.com/javase/tutorial/reflect/)).
2. It can be used only if it's the root node of your `XML`layout.

### How is implemented?

Analysing the source code of this class is very straightforward. It's composed of the following elements:

- Its parent class is a basic `FrameLayout` (sweet! my favourite one! :P)
- It uses a `Handler` to receive messages posted at a fixed rate of 500ms and where it decides if it should draw the content or not.
- Nothing more! Easy as a piece of cake!

Here is the complete source code:

```java
  private static class BlinkLayout extends FrameLayout {
        private static final int BLINK_DELAY = 500;
        private static final int MESSAGE_BLINK = 0x42;
        private final Handler mHandler;
        private boolean mBlink;
        private boolean mBlinkState;

        public BlinkLayout(Context context, AttributeSet attrs) {
            super(context, attrs);
            mHandler = new Handler(new Handler.Callback() {
                @Override
                public boolean handleMessage(Message msg) {
                    if (msg.what == MESSAGE_BLINK) {
                        if (mBlink) {
                            mBlinkState = !mBlinkState;
                            makeBlink();
                        }
                        invalidate();
                        return true;
                    }
                    return false;
                }
            });
        }

        @Override
        protected void dispatchDraw(Canvas canvas) {
            if (mBlinkState) {
                super.dispatchDraw(canvas);
            }
        }

        private void makeBlink() {
            Message message = mHandler.obtainMessage(MESSAGE_BLINK);
            mHandler.sendMessageDelayed(message, BLINK_DELAY);
        }

        @Override
        protected void onAttachedToWindow() {
            super.onAttachedToWindow();
            mBlink = true;
            mBlinkState = true;
            makeBlink();
        }

        @Override
        protected void onDetachedFromWindow() {
            super.onDetachedFromWindow();
            mBlink = false;
            mBlinkState = true;
            mHandler.removeMessages(MESSAGE_BLINK);
        }
    }
```

## BlinkLayout as a library

As `BlinkLayout` is a serious candidate of going to the hell in future revisions of the framework, I decided to create a library to preserve its history (yep, you can blame me if you see an adventurous developer use this)!

As a bonus I improved a little bit the code with the following features:

- You can configure the blink time. Yeeehaaa! Do you prefer a slow pace blink? Or do you prefer a quick blink to cause epilepsy? It's up to you!
- You can instantiate it programmatically or inside in another view. It's just another regular view.
- Mmmm… and nothing more! What do you expect?

Download the sample application in Google Play to get a sense of the feeling of getting a headache in real time:

<a href="https://play.google.com/store/apps/details?id=com.viewsforandroid.blinklayout.sample">
  <img alt="Get it on Google Play"
       src="http://developer.android.com/images/brand/en_generic_rgb_wo_45.png" />
</a>

Or go directly to the [Github repository](https://github.com/ViewsForAndroid/BlinkLayout) to fork it and start contributing (are you going to waste your time in this, seriously? Please don't do that!)!

## Conclusions

I don't know what was the intention of putting this kind of view in the `LayoutInflater` class, maybe for debugging purposes or maybe not… who knows!

Jokes apart, it was very fun to find this *surprise* in the source code of Android. It's one of the beauty of the Open Source movement: you can look, learn and … laugh!