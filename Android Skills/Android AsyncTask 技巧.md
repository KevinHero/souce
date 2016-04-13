---
title: Android AsyncTask 技巧
date: 2014-11-07 23:23:14
tags: AsyncTask
categories: 技术
---


#### AsyncTask的原理

其实AsyncTask的原理简单来说，就是：
- 一个任务队：用于存放自定义的（WorkerRunnable）。
- 一个线程池：初始化好任务队列，放入该线程池中。
- 一个内部Handler：用于提供线程池执行线程时与主线程之间的交互（刷新控件各种）。

<!--more-->
AsyncTask.execute(…)时，会启动AsyncTask的线程池，开始执行池中的任务队。只有doInBackground()在线程池中的线程执行，所以不能更新主UI线程的控件，剩下的onProgressUpdate()、onPostExecute()、onCancelled()、onPreExecute()在主UI线程中。onPreExecute()比较特殊的地方就是，它是主线程自己调用的。

这是execute()就直接走到的executeOnExecutor()源码：

    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }



其他的onProgressUpdate()、onPostExecute()、onCancelled()都经由InternalHandler调用。

    private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }


这里就是各种sendMessage(…)后的转发，最明显是是看到了onProgressUpdate()；

然后onPostExecute()、onCancelled()就在finish()中再次转发

    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }



综上，就是内置一个线程池既花式又完美的sendMessage(…)给内部Handler处理的二次封装类
AsyncTask 模板

#### ProgressBarAsyncTask：

    /**
     * Description：
     * Created by：CaMnter
     * Time：2015-09-17 14:19
     */
    public class ProgressBarAsyncTask extends AsyncTask<String, Integer, String> {

        private TextView textview;
        private ProgressBar progressBar;

        public ProgressBarAsyncTask(ProgressBar progressBar, TextView textview) {
            super();
            this.textview = textview;
            this.progressBar = progressBar;

        }

        /**
         * 对应AsyncTask第一个参数
         * 异步操作，不在主UI线程中，不能对控件进行修改
         * 可以调用publishProgress方法中转到onProgressUpdate(这里完成了一个handler.sendMessage(...)的过程)
         *
         * @param params The parameters of the task.
         * @return A result, defined by the subclass of this task.
         * @see #onPreExecute()
         * @see #onPostExecute
         * @see #publishProgress
         */
        @Override
        protected String doInBackground(String... params) {
            int i = 0;
            for (; i < 100; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                this.publishProgress(i);
            }
            return i + params[0];
        }

        /**
         * 对应AsyncTask第二个参数
         * 在doInBackground方法当中，每次调用publishProgress方法都会中转(handler.sendMessage(...))到onProgressUpdate
         * 在主UI线程中，可以对控件进行修改
         *
         * @param values The values indicating progress.
         * @see #publishProgress
         * @see #doInBackground
         */
        @Override
        protected void onProgressUpdate(Integer... values) {
            int value = values[0];
            this.progressBar.setProgress(value);
            this.textview.setText(value+"%");
        }

        /**
         * 对应AsyncTask第三个参数 (接受doInBackground的返回值)
         * 在doInBackground方法执行结束之后在运行，此时已经回来主UI线程当中 能对UI控件进行修改
         *
         * @param s The result of the operation computed by {@link #doInBackground}.
         * @see #onPreExecute
         * @see #doInBackground
         * @see #onCancelled(Object)
         */
        @Override
        protected void onPostExecute(String s) {
            this.textview.setText("执行结束：" + s);
        }

        /**
         * 运行在主UI线程中，此时是预执行状态，下一步是doInBackground
         *
         * @see #onPostExecute
         * @see #doInBackground
         */
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
        }

        /**
         * <p>Applications should preferably override {@link #onCancelled(Object)}.
         * This method is invoked by the default implementation of
         * {@link #onCancelled(Object)}.</p>
         * <p/>
         * <p>Runs on the UI thread after {@link #cancel(boolean)} is invoked and
         * {@link #doInBackground(Object[])} has finished.</p>
         *
         * @see #onCancelled(Object)
         * @see #cancel(boolean)
         * @see #isCancelled()
         */
        @Override
        protected void onCancelled() {
            super.onCancelled();
        }

    }



#### Activity：

    ProgressBarAsyncTask asyncTask = new ProgressBarAsyncTask(this.progressBar, this.textview);
    asyncTask.execute("%");