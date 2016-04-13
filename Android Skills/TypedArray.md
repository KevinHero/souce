---
title: 解析：TypedArray 为什么需要调用recycle()
date: 2014-07-18 02:21:14
tags: TypedArray
categories: 技术
---


在 Android 自定义 View 的时候，需要使用 TypedArray 来获取 XML layout 中的属性值，使用完之后，需要调用 recyle() 方法将 TypedArray 回收。

那么问题来了，这个TypedArray是个什么东西？为什么需要回收呢？TypedArray并没有占用IO，线程，它仅仅是一个变量而已，为什么需要 recycle？
为了解开这个谜，首先去找官网的 Documentation，到找 TypedArray 方法，得到下面一个简短的回答：

![](http://i.imgur.com/0YmjniV.png)

<!--more-->
这里写图片描述

告诉我们在确定使用完之后调用 recycle() 方法。于是进一步查看该方法的解释，如下：
这里写图片描述

简单翻译下来，就是说：回收 TypedArray,用于后续调用时可复用之。当调用该方法后，不能再操作该变量。

同样是一个简洁的答复，但没有解开我们心中的疑惑，这个TypedArray背后，到底隐藏着怎样的秘密……

求之不得，辗转反侧，于是我们决定深入源码，一探其究竟……


首先，是 TypedArray 的常规使用方法：

	TypedArray array = context.getTheme().obtainStyledAttributes(attrs,
	                R.styleable.PieChart,0,0);
	try {
	    mShowText = array.getBoolean(R.styleable.PieChart_showText,false);
	    mTextPos = array.getInteger(R.styleable.PieChart_labelPosition,0);
	}finally {
	    array.recycle();
	}

可见，TypedArray不是我们new出来的，而是调用了 obtainStyledAttributes 方法得到的对象，该方法实现如下：

	public TypedArray obtainStyledAttributes(AttributeSet set,
	                int[] attrs, int defStyleAttr, int defStyleRes) {
	    final int len = attrs.length;
	    final TypedArray array = TypedArray.obtain(Resources.this, len);
	    // other code .....
	    return array;
	}

我们只关注当前待解决的问题，其他的代码忽略不看。从上面的代码片段得知，TypedArray也不是它实例化的，而是调用了TypedArray的一个静态方法，得到一个实例，再做一些处理，最后返回这个实例。看到这里，我们似乎知道了什么，，，带着猜测，我们进一步查看该静态方法的内部实现：

	/**
	 * Container for an array of values that were retrieved with
	 * {@link Resources.Theme#obtainStyledAttributes(AttributeSet, int[], int, int)}
	 * or {@link Resources#obtainAttributes}.  Be
	 * sure to call {@link #recycle} when done with them.
	 *
	 * The indices used to retrieve values from this structure correspond to
	 * the positions of the attributes given to obtainStyledAttributes.
	 */
	public class TypedArray {

	    static TypedArray obtain(Resources res, int len) {
	        final TypedArray attrs = res.mTypedArrayPool.acquire();
	        if (attrs != null) {
	            attrs.mLength = len;
	            attrs.mRecycled = false;

	            final int fullLen = len * AssetManager.STYLE_NUM_ENTRIES;
	            if (attrs.mData.length >= fullLen) {
	                return attrs;
	            }

	            attrs.mData = new int[fullLen];
	            attrs.mIndices = new int[1 + len];
	            return attrs;
	        }

	        return new TypedArray(res,
	                new int[len*AssetManager.STYLE_NUM_ENTRIES],
	                new int[1+len], len);
	    }
	    // Other members ......
	}

仔细看一下这个方法的实现，我想大部分人都明了了，该类没有公共的构造函数，只提供静态方法获取实例，显然是一个典型的单例模式。在代码片段的第 13 行，很清晰的表达了这个 array 是从一个 array pool的池中获取的。

因此，我们得出结论：

程序在运行时维护了一个 TypedArray的池，程序调用时，会向该池中请求一个实例，用完之后，调用 recycle() 方法来释放该实例，从而使其可被其他模块复用。

那为什么要使用这种模式呢？答案也很简单，TypedArray的使用场景之一，就是上述的自定义View，会随着 Activity的每一次Create而Create，因此，需要系统频繁的创建array，对内存和性能是一个不小的开销，如果不使用池模式，每次都让GC来回收，很可能就会造成OutOfMemory。

这就是使用池+单例模式的原因，这也就是为什么官方文档一再的强调：使用完之后一定 recycle,recycle,recycle。
