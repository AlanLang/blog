---
title: 自定义停留时间的Toast
tags: Android
categories: Android
abbrlink: 21309
date: 2018-03-13 12:49:23
---
```java
/**
 * 弹出一个Toast
 * @param msg toast显示的消息
 * @param cnt 持续的时间（秒）
 */
public void showMyToast(String msg, final int cnt) {
    final Toast toast=Toast.makeText(MainActivity.this,msg, Toast.LENGTH_LONG);
    final Timer timer =new Timer();
    timer.schedule(new TimerTask() {
        @Override
        public void run() {
            toast.show();
        }
    },0,3000);
    new Timer().schedule(new TimerTask() {
        @Override
        public void run() {
            toast.cancel();
            timer.cancel();
        }
    }, cnt );
}
```