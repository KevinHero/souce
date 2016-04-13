---
title: 下拉刷新
date: 2014-05-17 22:01:14
tags: 下拉刷新
categories: 技术
---




最近在研究Android的下拉刷新功能，看了几个别人写的自定义控件：

1.android-pulltorefresh：https://github.com/chrisbanes/Android-PullToRefresh
2.android-pulltorefresh-listview：https://github.com/johannilsson/android-pulltorefresh

<!--more-->

前者是：一个强大的拉动刷新开源项目，支持各种控件下拉刷新，ListView、ViewPager、WevView、
ExpandableListView、GridView、ScrollView、Horizontal ScrollView、Fragment上下左右拉动刷新
并且它实现的下拉刷新ListView在item不足一屏情况下也不会显示刷新提示，体验更好。

后者是：下拉刷新ListView




这里我为什么特意例举这两个自定义控件呢?因为这两种自定义控件就单单从实现ListView的下拉刷
新方面而已，原理是不一样的!

前者的原理：简单而言就是自定义一个View的方式，然后我们就用这个View，不用ListView。但是这个
View，不是ListView，你想getCount()、addHeaderView()等都没有实现。

后者的原理：就是覆写ListView的部分方法（onScroll()、onScrollStateChanged()、onTouchEvent()、
setOnScrollListener()、setAdapter()等等）。




我个人感觉前者足够强大，但是没有达到我想要的效果，所以参照了另外一个大神的博客，然后自己照
着写了一遍代码，感触颇深。


主要思路：就是自定义一个View，这个View就相当一个包装袋一样，包裹商品（LIstView），然后
利用事件的覆盖（View覆写ListView对应的事件），间接屏蔽了ListView的事件。所以部分事件只
会在包装袋上触发，不会在ListView上触发。

优点：这样ListView就和事件的耦合度大大降低了。我们通过自定义的View去实现事件覆盖，触发了
View上的事件，通过这个View上对对应事件的监听间接修改ListView的Adapter对应的List数据。这
样我们在ListView上的操作少之又少，操作的是List<String,Object> ......而已。




PullToRefreshView：

    package com.zyy.android_csdn.pulltorefresh;

    import com.zyy.android_csdn.R;

    import android.annotation.SuppressLint;
    import android.content.Context;
    import android.content.SharedPreferences;
    import android.os.AsyncTask;
    import android.preference.PreferenceManager;
    import android.util.AttributeSet;
    import android.view.LayoutInflater;
    import android.view.MotionEvent;
    import android.view.View;
    import android.view.View.OnTouchListener;
    import android.view.ViewConfiguration;
    import android.view.animation.RotateAnimation;
    import android.widget.ImageView;
    import android.widget.LinearLayout;
    import android.widget.ListView;
    import android.widget.ProgressBar;
    import android.widget.TextView;

    /**
     * 下拉刷新控件(View)
     *
     * @author CaMnter
     *
     */
    public class PullToRefreshView extends LinearLayout implements OnTouchListener {

        /**
         * 下拉状态
         */
        public static final int STATUS_PULL_TO_REFRESH = 0;

        /**
         * 释放立即刷新状态
         */
        public static final int STATUS_RELEASE_TO_REFRESH = 1;

        /**
         * 正在刷新状态
         */
        public static final int STATUS_REFRESHING = 2;

        /**
         * 刷新完成或未刷新状态
         */
        public static final int STATUS_REFRESH_FINISHED = 3;

        /**
         * 下拉头部回滚的速度
         */
        public static final int SCROLL_SPEED = -20;

        /**
         * 一分钟的毫秒值，用于判断上次的更新时间
         */
        public static final long ONE_MINUTE = 60 * 1000;

        /**
         * 一小时的毫秒值，用于判断上次的更新时间
         */
        public static final long ONE_HOUR = 60 * ONE_MINUTE;

        /**
         * 一天的毫秒值，用于判断上次的更新时间
         */
        public static final long ONE_DAY = 24 * ONE_HOUR;

        /**
         * 一月的毫秒值，用于判断上次的更新时间
         */
        public static final long ONE_MONTH = 30 * ONE_DAY;

        /**
         * 一年的毫秒值，用于判断上次的更新时间
         */
        public static final long ONE_YEAR = 12 * ONE_MONTH;

        /**
         * 上次更新时间的字符串常量，用于作为SharedPreferences的键值
         */
        private static final String UPDATED_AT = "updated_at";

        /**
         * 下拉刷新的回调接口
         */
        private PullToRefreshListener mListener;

        /**
         * 用于存储上次更新时间
         */
        private SharedPreferences preferences;

        /**
         * 下拉头的View
         */
        private View header;

        /**
         * 需要去下拉刷新的ListView
         */
        private ListView listView;

        /**
         * 刷新时显示的进度条
         */
        private ProgressBar progressBar;

        /**
         * 指示下拉和释放的箭头
         */
        private ImageView arrow;

        /**
         * 指示下拉和释放的文字描述
         */
        private TextView description;

        /**
         * 上次更新时间的文字描述
         */
        private TextView updateAt;

        /**
         * 下拉头的布局参数
         */
        private MarginLayoutParams headerLayoutParams;

        /**
         * 上次更新时间的毫秒值
         */
        private long lastUpdateTime;

        /**
         * 为了防止不同界面的下拉刷新在上次更新时间上互相有冲突，使用id来做区分
         */
        private int mId = -1;

        /**
         * 下拉头的高度
         */
        private int hideHeaderHeight;

        /**
         * 当前处理什么状态，可选值有STATUS_PULL_TO_REFRESH, STATUS_RELEASE_TO_REFRESH,
         * STATUS_REFRESHING 和 STATUS_REFRESH_FINISHED
         */
        private int currentStatus = STATUS_REFRESH_FINISHED;;

        /**
         * 记录上一次的状态是什么，避免进行重复操作
         */
        private int lastStatus = currentStatus;

        /**
         * 手指按下时的屏幕纵坐标
         */
        private float yDown;

        /**
         * 在被判定为滚动之前用户手指可以移动的最大值
         */
        private int touchSlop;

        /**
         * 是否已加载过一次layout，这里onLayout中的初始化只需加载一次
         */
        private boolean loadOnce;

        /**
         * 当前是否可以下拉，只有ListView滚动到头的时候才允许下拉
         */
        private boolean ableToPull;

        /**
         * head View 包含的一个 LinearLayout
         */
        private LinearLayout pull_to_refresh_linearLayout;

        /**
         * 下拉刷新控件的构造函数，会在运行时动态添加一个下拉头的布局。
         *
         * @param context
         * @param attrs
         */
        @SuppressLint("InflateParams")
        public PullToRefreshView(Context context, AttributeSet attrs) {

            super(context, attrs);

            // 实例化SharedPreferences
            this.preferences = PreferenceManager
                    .getDefaultSharedPreferences(context);

            // 实例化 下拉头的 布局View
            this.header = LayoutInflater.from(context).inflate(
                    R.layout.pull_to_refresh, null, true);

            // 实例化 下拉头的 LinearLayout
            this.pull_to_refresh_linearLayout = (LinearLayout) this.header
                    .findViewById(R.id.pull_to_refresh_linearLayout);

            // 取得 下拉头的 布局 里的 组件
            this.progressBar = (ProgressBar) this.header
                    .findViewById(R.id.progress_bar);

            this.arrow = (ImageView) this.header.findViewById(R.id.arrow);

            this.description = (TextView) this.header
                    .findViewById(R.id.description);

            this.updateAt = (TextView) this.header.findViewById(R.id.updated_at);

            // 取得移动的最小距离,只有大于这个距离,才可认为是 move
            this.touchSlop = ViewConfiguration.get(context).getScaledTouchSlop();

            // 设置上次更新文字描述s
            this.refreshUpdatedAtValue();

            // 设置布局为垂直
            setOrientation(VERTICAL);

            // 下拉头的View 置顶
            addView(this.header, 0);

        }

        /**
         * 进行一些关键性的初始化操作
         *
         * 比如：将下拉头向上偏移进行隐藏，给ListView注册touch事件。
         */
        @Override
        protected void onLayout(boolean changed, int l, int t, int r, int b) {

            super.onLayout(changed, l, t, r, b);

            // 如果loadOnce没有设置(没有加载过一次layout)
            if (changed && !this.loadOnce) {

                this.hideHeaderHeight = -this.header.getHeight();

                this.headerLayoutParams = (MarginLayoutParams) this.header
                        .getLayoutParams();

                this.headerLayoutParams.topMargin = this.hideHeaderHeight;

                // getChildAt(1); 是 1 不是 L
                this.listView = (ListView) getChildAt(1);

                System.out.println("this.listView = " + this.listView);

                // 设置ListView的触摸事件
                this.listView.setOnTouchListener(this);

                // 更变 loadOnce 的值
                this.loadOnce = true;

            }

        }

        public static boolean result = false;

        public static boolean state = false;

        /**
         * 当ListView被触摸时调用，其中处理了各种下拉刷新的具体逻辑。
         */
        @SuppressLint("ClickableViewAccessibility")
        @Override
        public boolean onTouch(View v, MotionEvent event) {

            /**
             * 设定 {@link #ableToPull}
             */
            this.setIsAbleToPull(event);

            // 如果允许下拉
            if (this.ableToPull) {

                switch (event.getAction()) {

                // "按下"
                case MotionEvent.ACTION_DOWN:

                    PullToRefreshView.result = false;

                    PullToRefreshView.state = false;

                    this.yDown = event.getRawY();

                    break;

                case MotionEvent.ACTION_MOVE:

                    if (!PullToRefreshView.state) {

                        PullToRefreshView.result = true;

                        float yMove = event.getRawY();

                        int distance = (int) (yMove - this.yDown);

                        /**
                         * 如果手指是下滑状态，并且下拉头是完全隐藏的
                         *
                         * 就屏蔽下拉事件
                         */
                        if (distance <= 0
                                && this.headerLayoutParams.topMargin <= this.hideHeaderHeight) {

                            PullToRefreshView.result = false;

                            return false;

                        }

                        // 如果滑动的距离 小于 算是滑动的距离
                        if (distance < touchSlop) {

                            PullToRefreshView.result = false;

                            return false;

                        }

                        // 当前状态 不是 正在刷新
                        if (this.currentStatus != PullToRefreshView.STATUS_REFRESHING) {

                            if (this.headerLayoutParams.topMargin > 0) {

                                this.currentStatus = PullToRefreshView.STATUS_RELEASE_TO_REFRESH;

                            } else {

                                this.currentStatus = PullToRefreshView.STATUS_PULL_TO_REFRESH;

                            }

                            // 滑动时 如果 head部分 不可见
                            if (this.pull_to_refresh_linearLayout.getVisibility() == View.INVISIBLE) {

                                // 设置 可见 head部分
                                this.pull_to_refresh_linearLayout
                                        .setVisibility(View.VISIBLE);

                            }

                            // 通过偏移下拉头的topMargin值，来实现下拉效果
                            this.headerLayoutParams.topMargin = (distance / 2)
                                    + this.hideHeaderHeight;

                            this.header.setLayoutParams(this.headerLayoutParams);

                        }
                    }
                    break;

                case MotionEvent.ACTION_UP:

                default:

                    // 如果 当前状态 为 释放立即刷新状态
                    if (this.currentStatus == PullToRefreshView.STATUS_RELEASE_TO_REFRESH) {

                        // 松手时如果是释放立即刷新状态，就去调用正在刷新的任务
                        new RefreshingTask().execute();

                        // 如果 当前状态 为下拉状态
                    } else if (this.currentStatus == PullToRefreshView.STATUS_PULL_TO_REFRESH) {

                        // 松手时如果是下拉状态，就去调用隐藏下拉头的任务
                        new HideHeaderTask().execute();

                    }

                    break;
                }

                /*
                 * 时刻记得更新下拉头中的信息
                 *
                 * 如果 当前状态 为下拉状态 or 释放立即刷新状态
                 */
                if (this.currentStatus == PullToRefreshView.STATUS_PULL_TO_REFRESH
                        || this.currentStatus == PullToRefreshView.STATUS_RELEASE_TO_REFRESH) {

                    // 更新下拉头中的信息
                    this.updateHeaderView();

                    // 要让ListView失去焦点，否则被点击的那一项会一直处于选中状态
                    this.listView.setPressed(false);

                    this.listView.setFocusable(false);

                    this.listView.setFocusableInTouchMode(false);

                    this.lastStatus = this.currentStatus;

                    // 当前正处于下拉或释放状态，通过返回true屏蔽掉ListView的滚动事件

                    return true;

                }

            }

            return false;
        }

        /**
         * 给下拉刷新控件注册一个监听器。
         *
         * @param listener
         *            监听器的实现。
         * @param id
         *            为了防止不同界面的下拉刷新在上次更新时间上互相有冲突， 请不同界面在注册下拉刷新监听器时一定要传入不同的id。
         */
        public void setOnRefreshListener(PullToRefreshListener listener, int id) {

            this.mListener = listener;

            this.mId = id;

        }

        /**
         * 当所有的刷新逻辑完成后，记录调用一下
         *
         * 否则ListView将一直处于正在刷新状态。
         */
        public void finishRefreshing() {

            this.currentStatus = PullToRefreshView.STATUS_REFRESH_FINISHED;

            this.preferences.edit()
                    .putLong(UPDATED_AT + mId, System.currentTimeMillis()).commit();

            // 调用隐藏下拉头的任务
            new HideHeaderTask().execute();
        }

        /**
         * 根据当前ListView的滚动状态来设定 {@link #ableToPull}的值
         *
         * 每次都需要在onTouch中第一个执行
         *
         * 这样可以判断出当前应该是滚动ListView，还是应该进行下拉。
         *
         * @param event
         */
        private void setIsAbleToPull(MotionEvent event) {

            View firstChild = this.listView.getChildAt(0);

            if (firstChild != null) {

                int firstVisiblePos = this.listView.getFirstVisiblePosition();

                if (firstVisiblePos == 0 && firstChild.getTop() == 0) {

                    if (!this.ableToPull) {

                        this.yDown = event.getRawY();

                    }

                    /*
                     * 如果首个元素的上边缘，距离父布局值为0
                     *
                     * 就说明ListView滚动到了最顶部，此时应该允许下拉刷新
                     */
                    this.ableToPull = true;

                } else {

                    if (this.headerLayoutParams.topMargin != this.hideHeaderHeight) {

                        this.headerLayoutParams.topMargin = this.hideHeaderHeight;

                        this.header.setLayoutParams(this.headerLayoutParams);

                    }

                    this.ableToPull = false;

                }

            } else {

                // 如果ListView中没有元素，也应该允许下拉刷新
                this.ableToPull = true;

            }

        }

        /**
         * 更新下拉头中的信息
         */
        private void updateHeaderView() {

            // 如果上次的状态 不等于 这次的状态
            if (this.lastStatus != this.currentStatus) {

                // "下拉刷新状态"
                if (this.currentStatus == PullToRefreshView.STATUS_PULL_TO_REFRESH) {

                    this.description.setText(getResources().getString(
                            R.string.pull_to_refresh));

                    this.arrow.setVisibility(View.VISIBLE);

                    this.progressBar.setVisibility(View.GONE);

                    // 根据当前的状态来旋转箭头
                    this.rotateArrow();

                    // "释放立即刷新状态"
                } else if (this.currentStatus == PullToRefreshView.STATUS_RELEASE_TO_REFRESH) {

                    this.description.setText(getResources().getString(
                            R.string.release_to_refresh));

                    this.arrow.setVisibility(View.VISIBLE);

                    this.progressBar.setVisibility(View.GONE);

                    // 根据当前的状态来旋转箭头
                    this.rotateArrow();

                    // "正在刷新状态"
                } else if (this.currentStatus == PullToRefreshView.STATUS_REFRESHING) {

                    this.description.setText(getResources().getString(
                            R.string.refreshing));

                    this.progressBar.setVisibility(View.VISIBLE);

                    this.arrow.clearAnimation();

                    this.arrow.setVisibility(View.GONE);

                }

                // 刷新下拉头中上次更新时间的文字描述
                this.refreshUpdatedAtValue();

            }

        }

        /**
         * 根据当前的状态来旋转箭头
         */
        private void rotateArrow() {

            float pivotX = arrow.getWidth() / 2f;

            float pivotY = arrow.getHeight() / 2f;

            float fromDegrees = 0f;

            float toDegrees = 0f;

            // "下拉刷新状态"
            if (this.currentStatus == PullToRefreshView.STATUS_PULL_TO_REFRESH) {

                fromDegrees = 180f;

                toDegrees = 360f;

                // "释放立即刷新状态"
            } else if (this.currentStatus == PullToRefreshView.STATUS_RELEASE_TO_REFRESH) {

                fromDegrees = 0f;

                toDegrees = 180f;

            }

            // 旋转动画
            RotateAnimation animation = new RotateAnimation(fromDegrees, toDegrees,
                    pivotX, pivotY);

            animation.setDuration(100);

            animation.setFillAfter(true);

            arrow.startAnimation(animation);

        }

        /**
         * 刷新下拉头中上次更新时间的文字描述
         */
        private void refreshUpdatedAtValue() {

            // 从 SharedPreferences 取出 上次更新时间
            this.lastUpdateTime = this.preferences.getLong(
                    PullToRefreshView.UPDATED_AT + this.mId, -1);

            // 取得 以毫秒为单位的当前时间
            long currentTime = System.currentTimeMillis();

            // 计算时间差
            long timePassed = currentTime - this.lastUpdateTime;

            long timeIntoFormat;

            String updateAtValue;

            if (this.lastUpdateTime == -1) {

                updateAtValue = getResources().getString(R.string.not_updated_yet);

            } else if (timePassed < 0) {

                updateAtValue = getResources().getString(R.string.time_error);

            } else if (timePassed < PullToRefreshView.ONE_MINUTE) {

                updateAtValue = getResources().getString(R.string.updated_just_now);

            } else if (timePassed < PullToRefreshView.ONE_HOUR) {

                timeIntoFormat = timePassed / PullToRefreshView.ONE_MINUTE;

                String value = timeIntoFormat + "分钟";

                updateAtValue = String.format(
                        getResources().getString(R.string.updated_at), value);

            } else if (timePassed < PullToRefreshView.ONE_DAY) {

                timeIntoFormat = timePassed / PullToRefreshView.ONE_HOUR;

                String value = timeIntoFormat + "小时";

                updateAtValue = String.format(
                        getResources().getString(R.string.updated_at), value);

            } else if (timePassed < PullToRefreshView.ONE_MONTH) {

                timeIntoFormat = timePassed / PullToRefreshView.ONE_DAY;

                String value = timeIntoFormat + "天";

                updateAtValue = String.format(
                        getResources().getString(R.string.updated_at), value);

            } else if (timePassed < PullToRefreshView.ONE_YEAR) {

                timeIntoFormat = timePassed / PullToRefreshView.ONE_MONTH;

                String value = timeIntoFormat + "个月";

                updateAtValue = String.format(
                        getResources().getString(R.string.updated_at), value);

            } else {

                timeIntoFormat = timePassed / PullToRefreshView.ONE_YEAR;

                String value = timeIntoFormat + "年";

                updateAtValue = String.format(
                        getResources().getString(R.string.updated_at), value);

            }

            this.updateAt.setText(updateAtValue);

        }

        /**
         * 正在刷新的任务
         *
         * 在此任务中会去回调注册进来的下拉刷新监听器
         *
         * @author CaMnter
         *
         */
        class RefreshingTask extends AsyncTask<Void, Integer, Void> {

            @Override
            protected Void doInBackground(Void... params) {

                int topMargin = PullToRefreshView.this.headerLayoutParams.topMargin;

                while (true) {

                    topMargin = topMargin + PullToRefreshView.SCROLL_SPEED;

                    if (topMargin <= 0) {

                        topMargin = 0;

                        break;

                    }

                    // 每个调用此方法将触发onProgressUpdate在UI线程的执行
                    this.publishProgress(topMargin);

                    PullToRefreshView.this.sleep(10);

                }

                PullToRefreshView.this.currentStatus = PullToRefreshView.STATUS_REFRESHING;

                this.publishProgress(0);

                PullToRefreshView.this.sleep(3000);

                PullToRefreshView.this.finishRefreshing();

                return null;

            }

            @Override
            protected void onPostExecute(Void result) {

                /**
                 * 如果 doInBackground 的onRefresh() 中有事件处理 那么 执行不到 doInBackground 里的
                 * return null 就一直处在刷新状态效果 此时，可以把 doInBackground 里的 onRefresh操作换到
                 * onPostExecute里来
                 **/
                if (PullToRefreshView.this.mListener != null) {

                    // 调用刷新方法
                    PullToRefreshView.this.mListener.onRefresh();

                }

                super.onPostExecute(result);

            }

            @Override
            protected void onProgressUpdate(Integer... topMargin) {

                // 更新下拉头中的信息
                PullToRefreshView.this.updateHeaderView();

                PullToRefreshView.this.headerLayoutParams.topMargin = topMargin[0];

                PullToRefreshView.this.header
                        .setLayoutParams(PullToRefreshView.this.headerLayoutParams);

            }

        }

        /**
         * 隐藏下拉头的任务，当未进行下拉刷新或下拉刷新完成后
         *
         * 此任务将会使下拉头重新隐藏
         *
         * @author CaMnter
         *
         */
        class HideHeaderTask extends AsyncTask<Void, Integer, Integer> {

            @Override
            protected Integer doInBackground(Void... params) {

                int topMargin = PullToRefreshView.this.headerLayoutParams.topMargin;

                while (true) {

                    topMargin = topMargin + PullToRefreshView.SCROLL_SPEED;

                    if (topMargin <= PullToRefreshView.this.hideHeaderHeight) {

                        topMargin = PullToRefreshView.this.hideHeaderHeight;

                        break;

                    }

                    // 每个调用此方法将触发onProgressUpdate在UI线程的执行
                    this.publishProgress(topMargin);

                    PullToRefreshView.this.sleep(10);

                }

                return topMargin;

            }

            @Override
            protected void onProgressUpdate(Integer... topMargin) {

                PullToRefreshView.this.headerLayoutParams.topMargin = topMargin[0];

                PullToRefreshView.this.header
                        .setLayoutParams(PullToRefreshView.this.headerLayoutParams);

            }

            /**
             *
             * doInBackground后在UI线程上运行
             *
             * 指定的结果是doInBackground返回的值。
             *
             * 如果任务被取消,这个方法不会被调用。
             *
             */
            @Override
            protected void onPostExecute(Integer topMargin) {

                PullToRefreshView.this.headerLayoutParams.topMargin = topMargin;

                PullToRefreshView.this.header
                        .setLayoutParams(PullToRefreshView.this.headerLayoutParams);

                // 设置状态为 刷新结束
                PullToRefreshView.this.currentStatus = PullToRefreshView.STATUS_REFRESH_FINISHED;

            }

        }

        /**
         * 使当前线程睡眠指定的毫秒数
         *
         * @param time
         *            指定当前线程睡眠多久，以毫秒为单位
         */
        private void sleep(int time) {

            try {

                Thread.sleep(time);

            } catch (InterruptedException e) {

                e.printStackTrace();

            }

        }

        /**
         * 下拉刷新的监听器，使用下拉刷新的地方应该注册PullToRefreshListener
         *
         * 来获取刷新回调
         *
         * @author CaMnter
         */
        public interface PullToRefreshListener {

            /**
             * 刷新时会去回调此方法，在方法内编写具体的刷新逻辑。
             *
             * 注意此方法是在子线程中调用的，
             *
             * 可以不必另开线程来进行耗时操作。
             */
            void onRefresh();

        }

    }





温馨提示：这里做了if (this.pull_to_refresh_linearLayout.getVisibility() == View.INVISIBLE) 。设置
 可见 head的linearLayout部分，原因是因为Fragment的兼容问题。在Acticity里没什么问题；到了Fragment，
head部分都会闪一下再消失，我在布局里让LinearLayout部分不可见，然后通过滑动事件动态设置可见，取巧得躲避了这个问题。

还有一点：就是调用刷新方法这里：PullToRefreshView.this.mListener.onRefresh()；如果
doInBackground 的onRefresh() 中有事件处理，那么 执行不到  doInBackground 里的 return null ，
就一直处在刷新状态效果。此时，可以把 doInBackground 里的 onRefresh操作换到onPostExecute(Void result)
里来 。               

MOnCreateContextMenuListener：

    package com.zyy.android_csdn;

    import android.view.ContextMenu;
    import android.view.ContextMenu.ContextMenuInfo;
    import android.view.View;
    import android.view.View.OnCreateContextMenuListener;

    /**
     * 覆盖ListView长按事件
     *
     * @author CaMnter
     *
     */
    public class MOnCreateContextMenuListener implements
            OnCreateContextMenuListener {

        @Override
        public void onCreateContextMenu(ContextMenu menu, View v,
                ContextMenuInfo menuInfo) {

            if (!PullToRefreshView.result) {

                PullToRefreshView.state = true;

                menu.add(0, 0, 0, "你好");

                menu.add(0, 0, 0, "再见");

            }

        }

    }


RefreshActivity：

    package com.zyy.android_csdn;

    import java.util.ArrayList;
    import java.util.List;

    import android.annotation.SuppressLint;
    import android.app.Activity;
    import android.graphics.Bitmap;
    import android.graphics.BitmapFactory;
    import android.os.Bundle;
    import android.os.Handler;
    import android.os.Message;
    import android.view.View;
    import android.view.Window;
    import android.widget.ImageView;
    import android.widget.ListView;
    import android.widget.TextView;

    import com.zyy.android_csdn.pulltorefresh.MOnCreateContextMenuListener;
    import com.zyy.android_csdn.pulltorefresh.PullToRefreshView;
    import com.zyy.android_csdn.pulltorefresh.PullToRefreshView.PullToRefreshListener;
    import com.zyy.android_csdn.skill.ExquisiteAdapter;

    /**
     * "下拉刷新，你值得拥有"
     *
     * @author CaMnter
     *
     */
    @SuppressLint("HandlerLeak")
    public class RefreshActivity extends Activity {

        PullToRefreshView pullToRefreshView;

        private ListView listView;

        private List<String> data_list;

        private ExquisiteAdapter exquisiteAdapter;

        private Bitmap oneIcon;

        private Bitmap zeroIcon;

        TextView newText;

        ImageView newImage;

        public static final String NEW_TEXT = "		Save you from anything";

        View v;

        private Handler handler = new Handler() {

            @Override
            public void handleMessage(Message msg) {

                RefreshActivity.this.data_list.add(0, RefreshActivity.NEW_TEXT);

                RefreshActivity.this.exquisiteAdapter.notifyDataSetChanged();

            }

        };

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            requestWindowFeature(Window.FEATURE_NO_TITLE);
            setContentView(R.layout.refresh);

            this.data_list = new ArrayList<String>();

            for (int i = 0; i < 25; i++) {

                this.data_list.add("		" + i + "");

            }

            pullToRefreshView = (PullToRefreshView) findViewById(R.id.refreshable_view);

            this.listView = (ListView) super.findViewById(R.id.list_view);

            this.oneIcon = BitmapFactory.decodeResource(getResources(),
                    R.drawable.one);

            this.zeroIcon = BitmapFactory.decodeResource(getResources(),
                    R.drawable.zero);

            this.exquisiteAdapter = new ExquisiteAdapter(this, data_list, oneIcon,
                    zeroIcon);

            this.listView.setAdapter(this.exquisiteAdapter);

            pullToRefreshView.setOnRefreshListener(new PullToRefreshListener() {
                @Override
                public void onRefresh() {


                    Message msg = RefreshActivity.this.handler.obtainMessage(1);

                    RefreshActivity.this.handler.sendMessage(msg);



                }
            }, 0);

            listView.setOnCreateContextMenuListener(new MOnCreateContextMenuListener());
        }
    }




Activity显示的布局(refresh.xml)：

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin"
        tools:context="com.zyy.android_csdn.MainActivity" >

        <com.zyy.android_csdn.PullToRefreshView
            android:id="@+id/refreshable_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent" >

            <ListView
                android:id="@+id/list_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent" >

                <!-- android:scrollbars="none" -->

            </ListView>
        </com.zyy.android_csdn.PullToRefreshView>

    </RelativeLayout>


下拉头的View的(pull_to_refresh.xml)布局：

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/pull_to_refresh_head"
        android:layout_width="fill_parent"
        android:layout_height="60dip" >

        <LinearLayout
            android:id="@+id/pull_to_refresh_linearLayout"
            android:layout_width="200dip"
            android:layout_height="60dip"
            android:layout_centerInParent="true"
            android:orientation="horizontal"
            android:visibility="invisible" >

            <RelativeLayout
                android:layout_width="0dip"
                android:layout_height="60dip"
                android:layout_weight="3" >

                <ImageView
                    android:id="@+id/arrow"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_centerInParent="true"
                    android:src="@drawable/arrow" />

                <ProgressBar
                    android:id="@+id/progress_bar"
                    android:layout_width="30dip"
                    android:layout_height="30dip"
                    android:layout_centerInParent="true"
                    android:visibility="gone" />
            </RelativeLayout>

            <LinearLayout
                android:layout_width="0dip"
                android:layout_height="60dip"
                android:layout_weight="12"
                android:orientation="vertical" >

                <TextView
                    android:id="@+id/description"
                    android:layout_width="fill_parent"
                    android:layout_height="0dip"
                    android:layout_weight="1"
                    android:gravity="center_horizontal|bottom"
                    android:text="@string/pull_to_refresh" />

                <TextView
                    android:id="@+id/updated_at"
                    android:layout_width="fill_parent"
                    android:layout_height="0dip"
                    android:layout_weight="1"
                    android:gravity="center_horizontal|top"
                    android:text="@string/updated_at" />
            </LinearLayout>
        </LinearLayout>

    </RelativeLayout>


string.xml：

    <string name="pull_to_refresh">下拉可以刷新</string>
    <string name="release_to_refresh">释放立即刷新</string>
    <string name="refreshing">正在刷新…</string>
    <string name="not_updated_yet">暂未更新过</string>
    <string name="updated_at">上次更新于%1$s前</string>
    <string name="updated_just_now">刚刚更新</string>
    <string name="time_error">时间有问题</string>


效果图：

![](http://img.blog.csdn.net/20141118183653229?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTY0MzA3MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141118183704773?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTY0MzA3MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141118183715678?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTY0MzA3MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141118183726406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTY0MzA3MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
