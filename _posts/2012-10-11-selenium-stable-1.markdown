---
layout: post
title: "提高selenium自动化的稳定性1-点击空白区域"
date: 2012-10-11 14:12
comments: true
categories: selenium
tags: [ webdriver, selenium , stable  ]
---
在写selenium自动化的过程中，经常会遇到这样的问题：   
>1.在同一个页面内做操作，比如点击某个按钮后，弹出一个框，再点击另外一个按钮，又弹出一个框   
>2.此时如果第一个click操作后，第二个click再点击时，由于前一个弹出的框仍旧在前端显示，就会出错   
>3.在实际人工操作中，点击出第一个框后，点击一下空白区域，在点击出现第二个框。因此，可以考虑一个点击空白区域的方法

##实现方法如下    
    /**
	 * 点击空白区域：坐标（0，0）
	 */
	public static void clickBlankArea(WebDriver driver) {
		Actions actions = new Actions(driver);
		actions.moveByOffset(0, 0).click().build().perform();
	}

让driver先移动到一个空白位置（此处设为(0,0)坐标点），做一下点击操作即可
