---
layout: post
title: window的showAsDropDown失效的问题
description: window的showAsDropDown失效的问题
categories: android
keywords: window, showAsDropDown, android
---

在使用Popupwindow的showAsDropDown的时候，有时候会在特定机型上无法正确显示（全屏显示了）。  


目前的解决方案是如果需要全屏显示的popwindow，要计算出window的实际高度然后调用popupwindow的setHeight方法后再调用showAdDropDown即可。  
获取PopupWindow的实际高度: 
<pre>
    public static void showPopwindow(Context context, PopupWindow popupWindow, View anchor) {
        int[] locations = new int[2];
        anchor.getLocationOnScreen(locations);
        int screenHeight = context.getApplicationContext().getResources().getDisplayMetrics(); 
	int height = screenHeight - anchor.getHeight() - locations[1];
        popupWindow.setHeight(height);
        popupWindow.showAsDropDown(anchor);
    }
</pre>
