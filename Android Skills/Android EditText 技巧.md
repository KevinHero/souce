---
title: 图像、背景、View更新、布局、内存
date: 2014-05-05 22:01:14
tags: 图像、背景、View更新、布局、内存
categories: 技术
---



### 解决EditView抢焦点事件

在最外层布局加上

    android:focusable="true"
    android:focusableInTouchMode="true"

<!--more-->
实例

    <LinearLayout
            android:id="@+id/top_search"
            android:layout_width="fill_parent"
            android:layout_height="@dimen/button_bar_height"
            android:layout_gravity="top|center_horizontal"
            android:layout_marginTop="2dip"
            android:background="@drawable/bottom_bar_bg"
            android:focusable="true"
            android:focusableInTouchMode="true"
            android:gravity="center_horizontal" >

            <EditText
                android:id="@+id/url_edittext"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_marginLeft="4dip"
                android:layout_marginRight="4dip"
                android:layout_weight="3"
                android:drawablePadding="5.0dip"
                android:hint="请输入地址"
                android:singleLine="true"
                android:textSize="15dip" />

            <EditText
                android:id="@+id/search_edittext"
                android:layout_width="0dip"
                android:layout_height="wrap_content"
                android:layout_marginLeft="4dip"
                android:layout_marginRight="4dip"
                android:layout_weight="1"
                android:drawablePadding="5.0dip"
                android:hint="搜索"
                android:singleLine="true"
                android:textSize="15dip" />
    </LinearLayout>


### 将EditText的光标定位到字符的最后面

    public void setEditTextCursorLocationEnd(EditText editText) {
        CharSequence text = editText.getText();
        if (text instanceof Spannable) {
           Spannable spanText = (Spannable) text;
           Selection.setSelection(spanText, text.length());
        }
    }


### 添加下划线

	this.editText.getPaint().setFlags(Paint.UNDERLINE_TEXT_FLAG);



### 设置抗锯齿

	this.editText.getPaint().setAntiAlias(true);



### 焦点事件监听

    this.editText.setOnFocusChangeListener(new android.view.View.
                    OnFocusChangeListener() {
                @Override
                public void onFocusChange(View v, boolean hasFocus) {
                    if (hasFocus) {
                        // 得到焦点时


                    } else {
                        // 失去焦点时

                    }
                }
    });


### 手动获取焦点

    this.editText.setFocusable(true);
    this.editText.setFocusableInTouchMode(true);
    this.editText.requestFocus();


### 弹出软键盘

    public static void popSoftkeyboard(Context context,View view,boolean pop) {
            InputMethodManager imm = (InputMethodManager)           context.getSystemService(Context.INPUT_METHOD_SERVICE);
            if (pop) {
                view.requestFocus();
                imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT);
            } else {
                imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
            }
     }



### 限制长度

XML

在 xml 文件中设置文本编辑框属性作字符数限制

	如：android:maxLength=”10” 即限制最大输入字符个数为10

代码
在代码中使用InputFilter 进行过滤

    //即限定最大输入字符数为20
    this.editText.setFilters(new InputFilter[]{new InputFilter.LengthFilter(20)});
