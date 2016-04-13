---
title: CountDownTimer（倒数计时器）
date: 2014-06-01 22:01:14
tags: CountDownTimer
categories: 技术
---



  其实在很多时候，我们都需要一个倒计时的功能，这个功能我们自己可以根据java自带的TimerTask
去实现。这里，提到的是一个在基本Android开发书籍中都很少介绍到的一个Android原生自带倒数计
时器 - CountDownTimer 。

<!--more-->
接下来，我们通过CountDownTimer的源代码，看观赏一下它的实现原理：

/*
 * Copyright (C) 2008 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package android.os;

import android.util.Log;

/**
 * Schedule a countdown until a time in the future, with
 * regular notifications on intervals along the way.
 *
 * Example of showing a 30 second countdown in a text field:
 *
 * <pre class="prettyprint">
 * new CountDownTimer(30000, 1000) {
 *
 *     public void onTick(long millisUntilFinished) {
 *         mTextField.setText("seconds remaining: " + millisUntilFinished / 1000);
 *     }
 *
 *     public void onFinish() {
 *         mTextField.setText("done!");
 *     }
 *  }.start();
 * </pre>
 *
 * The calls to {@link #onTick(long)} are synchronized to this object so that
 * one call to {@link #onTick(long)} won't ever occur before the previous
 * callback is complete.  This is only relevant when the implementation of
 * {@link #onTick(long)} takes an amount of time to execute that is significant
 * compared to the countdown interval.
 */
public abstract class CountDownTimer {

    /**
     * Millis since epoch when alarm should stop.
     */
    private final long mMillisInFuture;

    /**
     * The interval in millis that the user receives callbacks
     */
    private final long mCountdownInterval;

    private long mStopTimeInFuture;

    /**
     * @param millisInFuture The number of millis in the future from the call
     *   to {@link #start()} until the countdown is done and {@link #onFinish()}
     *   is called.
     * @param countDownInterval The interval along the way to receive
     *   {@link #onTick(long)} callbacks.
     */
    public CountDownTimer(long millisInFuture, long countDownInterval) {
        mMillisInFuture = millisInFuture;
        mCountdownInterval = countDownInterval;
    }

    /**
     * Cancel the countdown.
     */
    public final void cancel() {
        mHandler.removeMessages(MSG);
    }

    /**
     * Start the countdown.
     */
    public synchronized final CountDownTimer start() {
        if (mMillisInFuture <= 0) {
            onFinish();
            return this;
        }
        mStopTimeInFuture = SystemClock.elapsedRealtime() + mMillisInFuture;
        mHandler.sendMessage(mHandler.obtainMessage(MSG));
        return this;
    }


    /**
     * Callback fired on regular interval.
     * @param millisUntilFinished The amount of time until finished.
     */
    public abstract void onTick(long millisUntilFinished);

    /**
     * Callback fired when the time is up.
     */
    public abstract void onFinish();


    private static final int MSG = 1;


    // handles counting down
    private Handler mHandler = new Handler() {

        @Override
        public void handleMessage(Message msg) {

            synchronized (CountDownTimer.this) {
                final long millisLeft = mStopTimeInFuture - SystemClock.elapsedRealtime();

                if (millisLeft <= 0) {
                    onFinish();
                } else if (millisLeft < mCountdownInterval) {
                    // no tick, just delay until done
                    sendMessageDelayed(obtainMessage(MSG), millisLeft);
                } else {
                    long lastTickStart = SystemClock.elapsedRealtime();
                    onTick(millisLeft);

                    // take into account user's onTick taking time to execute
                    long delay = lastTickStart + mCountdownInterval - SystemClock.elapsedRealtime();

                    // special case: user's onTick took more than interval to
                    // complete, skip to next interval
                    while (delay < 0) delay += mCountdownInterval;

                    sendMessageDelayed(obtainMessage(MSG), delay);
                }
            }
        }
    };
}


cancel() ：取消的倒计时。

start() ：开始倒计时。

onTick()：回调执行固定时间间隔。

onFinish() ：倒计时结束时

源代码中，我们可以看出 ：CountDownTimer类的同步start()方法执行后，做了一些简单
的时间判断和计算后(判断总时间、计算剩余时间)，然后发送到mHandler，在mHandler里
同步操作，然后又做了一些逻辑的运算和判断，为了设置onFinish()和onTick()方法的执行点
 然后 如果执行到了onTick的话，继续发送事件到mHandler。

就是start()->mHandler->mHandler->mHandler... 直到 mHandler中执行了onFinish()。


所以主要的操作，我们都放在onTick()和onFinish()方法中。

那么这里，给一个小小的实现类：
package com.zyy.android_csdn.skill;


import android.content.Context;
import android.graphics.drawable.Drawable;
import android.os.CountDownTimer;
import android.widget.Button;

/**
 *
 * 倒计时按钮计时器
 *
 * @author CaMnter
 *
 */
public class CountDownButtonTimer extends CountDownTimer {

	public static final int TIME_COUNT_FUTURE = 60000;
	public static final int TIME_COUNT_INTERVAL = 1000;

	// 用于存放 Context
	private Context mContext;

	// 用于存放 按钮
	private Button mButton;

	// 用于 存放 按钮Text
	private String mOriginalText;

	// 用于 存放 按钮背景
	private Drawable mOriginalBackground;

	// 用于 存放 按钮颜色
	private int mOriginalTextColor;

	private Drawable mTickBackground;

	public CountDownButtonTimer() {
		super(TIME_COUNT_FUTURE, TIME_COUNT_INTERVAL);
	}

	public CountDownButtonTimer(long millisInFuture, long countDownInterval) {
		super(millisInFuture, countDownInterval);
	}

	/**
	 *
	 * 初始化 Button及其相关内容
	 *
	 * @param context
	 * @param button
	 */
	public void init(Context context, Button button) {

		this.mContext = context;
		this.mButton = button;
		this.mOriginalText = mButton.getText().toString();
		this.mOriginalBackground = mButton.getBackground();
		this.mTickBackground = this.mOriginalBackground;
		this.mOriginalTextColor = mButton.getCurrentTextColor();

	}

	public void setTickDrawable(Drawable tickDrawable) {
		this.mTickBackground = tickDrawable;
	}

	/**
	 *
	 * 计时器结束的时
	 *
	 */
	@Override
	public void onFinish() {
		if (mContext != null && mButton != null) {
			mButton.setText(mOriginalText);
			mButton.setTextColor(mOriginalTextColor);
			mButton.setBackground(mOriginalBackground);
			mButton.setClickable(true);
		}

	}

	/**
	 *
	 * 倒计时开始时
	 *
	 */
	@Override
	public void onTick(long millisUntilFinished) {
		if (mContext != null && mButton != null) {
			mButton.setClickable(false);
			mButton.setBackground(mTickBackground);
			mButton.setTextColor(mContext.getResources().getColor(
					android.R.color.darker_gray));
			mButton.setText(millisUntilFinished / 1000 + " 秒后可重新获取验证码");
		}

	}

}
