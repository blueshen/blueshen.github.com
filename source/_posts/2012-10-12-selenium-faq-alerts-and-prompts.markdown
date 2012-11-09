---
layout: post
title: "selenium FAQ:如何处理JavaScript弹出的alert、prompt窗口"
date: 2012-10-12 14:39
comments: true
categories: selenium
tags: [ webdriver , selenium , alert , FAQ ]
---
##经常会碰到，页面操作后，出现一个alert窗口或者prompt确认窗口的情况，这时需要获得窗口的提示信息以及点击确定或取消的情况。
	// Get a handle to the open alert, prompt or confirmation
	Alert alert = driver.switchTo().alert();
	// Get the text of the alert or prompt
	alert.getText();  
	// And acknowledge the alert (equivalent to clicking "OK")
	alert.accept();
这是通用的处理方法。但是如果弹出的窗口不是`alert()`或者`prompt()`弹出来的则不适用。请注意。