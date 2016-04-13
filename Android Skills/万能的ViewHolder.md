---
title: ViewHolder的作用
date: 2014-05-19 22:01:14
tags: ViewHolder的作用
categories: 技术
---

Adapter的getVIew()以得到旧视图（convertView）：

    /**
     * Get a View that displays the data at the specified position in the data set. You can either
     * create a View manually or inflate it from an XML layout file. When the View is inflated, the
     * parent View (GridView, ListView...) will apply default layout parameters unless you use
     * {@link android.view.LayoutInflater#inflate(int, android.view.ViewGroup, boolean)}
     * to specify a root view and to prevent attachment to the root.
     * 
     * @param position The position of the item within the adapter's data set of the item whose view
     *        we want.
     * @param convertView The old view to reuse, if possible. Note: You should check that this view
     *        is non-null and of an appropriate type before using. If it is not possible to convert
     *        this view to display the correct data, this method can create a new view.
     *        Heterogeneous lists can specify their number of view types, so that this View is
     *        always of the right type (see {@link #getViewTypeCount()} and
     *        {@link #getItemViewType(int)}).
     * @param parent The parent that this view will eventually be attached to
     * @return A View corresponding to the data at the specified position.
     */
    View getView(int position, View convertView, ViewGroup parent);

<!--more-->

看过精致Adapter都知道，ListView的一个Item移出屏幕的范围，和一个Item进入屏幕的范围的时候，Adapter的getView(int position, View convertView, ViewGroup parent)里都能得到这个Item的View，也就是上述所说的旧视图（convertView） 。ListView的的Item如果样式都是一样的话，我们可以拿到旧视图的View，然后根据ViewId上的去取得组件并修改其内容，然后return该View。达到复用的效果。所以这里，涉及到了ViewHolder去保存该View上组件的各个ViewId，并且将ViewHolder继续保存到return的convertView的tag中，以供下一次拿到convertView时取得，规避了ViewHolder的不断创建，也实现了ViewHolder的复用。

其实findViewById()的性能是很低的，引入ViewHolder的目的也是为减少getView()时findViewById()的次数。

万能ViewHolder的原理

类似于内部存放了一个字典，用于根据ViewId得到View。

    public class ViewHolder {
        private final SparseArray<View> views;
        private View convertView;

        public ViewHolder(View convertView) {
            this.views = new SparseArray<>();
            this.convertView = convertView;
        }

        public <T extends View> T findViewById(int viewId) {
            View view = views.get(viewId);
            if (view == null) {
                view = convertView.findViewById(viewId);
                views.put(viewId, view);
            }
            return (T) view;
        }
    }



在你的BaseListAdapter中可以这么实现


    public abstract View getView(int position, View convertView, ViewHolder viewHolder);

    @Override
    public final View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        if (convertView == null) {
            convertView = mInflater.inflate(getItemLayout(), null);
            viewHolder = new ViewHolder(convertView);
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }
        try {
            return getView(position, convertView, viewHolder);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return convertView;

    }



这样的话可以让BaseListAdapter的子类强制继承getView然后与BaseListAdapter的getView对接后续操作。相当于执行了：BaseListAdapter.getView() -> BaseListAdapter子类.getView() 。