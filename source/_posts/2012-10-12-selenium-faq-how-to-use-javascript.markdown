---
layout: post
title: "selenium FAQ:怎么样调用Javascript？"
date: 2012-10-12 14:30
comments: true
categories: selenium
tags: [ webdriver, selenium, JavaScript ]
---
##selenium自动化开发中，难免需要用到直接调用javascript，怎么用呢？
	WebDriver driver; // Assigned elsewhere
	JavascriptExecutor js = (JavascriptExecutor) driver;
	js.executeScript("return document.title");
直接将driver强制转化为JavascriptExecutor,然后执行javascript即可。
