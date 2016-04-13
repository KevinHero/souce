---
title: String相关类性能测试
date: 2014-05-13 22:01:14
tags: String相关类性能测试
categories: 技术
---


String、StringBuffer、StringBuilder

    String： String是一个不可变的对象，每次对String类的改变实质都是新生成了一个新的String，然后把指针指向新String对象。每次生成对象都会对系统性能产生影响，速度会相当慢。

    StringBuffer： 在字符改变的时候，不会产生新的对象，线程安全的可变字符序列。

    StringBuilder： 同StringBuffer类似，线程不安全的可变字符序列。
<!--more-->
Strng相关类的性能测试

    private void stringRun(){
        long begin = System.currentTimeMillis();
        String string = "";
        for (int i = 0; i < COUNT; i++) {
            string += TEMP;
            Log.v("CaMnter","StringRun:  "+i);
        }
        long end = System.currentTimeMillis();
        this.stringTV.setText("StringRun: " + (end - begin) + " millis");
    }

    private void stringBufferRun(){
        long begin = System.currentTimeMillis();
        StringBuffer stringBuffer = new StringBuffer();
        for (int i = 0; i < COUNT; i++) {
            stringBuffer.append(TEMP);
            Log.v("CaMnter", "StringBufferRun:  " + i);
        }
        long end = System.currentTimeMillis();
        this.stringBufferTV.setText("StringBufferRun: " + (end - begin) +" millis");
    }

    private void stringBuilderRun(){
        long begin = System.currentTimeMillis();
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < COUNT; i++) {
            stringBuilder.append(TEMP);
            Log.v("CaMnter", "StringBuilderRun:  " + i);
        }
        long end = System.currentTimeMillis();
        this.stringBuilderTV.setText("StringBuilderRun: " + (end - begin) + " millis");
    }
---
    09-11 15:08:02.615 13748-13748/camnter.stringclassperformance V/CaMnter﹕
    StringRun: StringRun: 1240315 millis
    09-11 15:08:02.615 13748-13748/camnter.stringclassperformance V/CaMnter﹕ StringRun: StringBufferRun: 3467 millis
    09-11 15:08:02.615 13748-13748/camnter.stringclassperformance V/CaMnter﹕
    StringRun: StringBuilderRun: 3037 millis

关于String相关类的总结

    String的执行时间远远高于两者的执行时间
    单线程运行下，StringBuilder的效率比StringBuffer高，但是线程不安全
    涉及到多线程，用StringBuffer；单线程，用StringBuilder
