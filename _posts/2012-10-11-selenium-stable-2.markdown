---
layout: post
title: "提高selenium自动化的稳定性2-等待"
date: 2012-10-11 14:32
comments: true
categories: selenium
tags: [ webdriver , selenium , stable, wait  ]
---
##很多页面元素都是ajax动态生成的，这就要求进行适当的等待##
##如何进行等待呢？##
###1.直接sleep###
	public static void sleep(int seconds) {
		try {
			TimeUnit.SECONDS.sleep(seconds);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
这种方法，用于直接的让thread进行等待指定的seconds。  
<!--more-->
###2.使用selenium webdriver提供的等待方法
	driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
这种方法是webdriver提供的一种隐式等待。  
隐式等待是指当要查找元素（`driver.findElement()`），而这个元素没有马上出现时，告诉WebDriver查询Dom一定时间。默认值是0,但是设置之后，这个时间将在WebDriver对象实例整个生命周期都起作用。   
比如使用：`driver.findElement(By.id("element"));`来查找id="element"的元素。如果没有设置隐式等待，那么执行到这一步的时候就会直接报错`NoSuchElementException`。而设置后，则会在10秒内不断就查询元素是否存在，如果存在则返回。超过10秒仍没找到，才报错。
###3.使用WebDriver提供的`Wait<T>`接口

	FluentWait<T> implements Wait<T>  

	Wait<WebDriver> wait = new FluentWait<WebDriver>(driver)
       .withTimeout(30, SECONDS)
       .pollingEvery(5, SECONDS)
       .ignoring(NoSuchElementException.class);

   	WebElement element = wait.until(new Function<WebDriver, WebElement>() {
     public WebElement apply(WebDriver driver) {
       return driver.findElement(By.id("element"));
     }
   	});
	element.click();//something to do
此方法用于等待一个元素在页面上出现，超时时间为30S，每隔5S去请求一次，并且忽略掉until中抛出的`NoSuchElementException`。   
FluentWait的源码中这样写到:    

	private Duration timeout = FIVE_HUNDRED_MILLIS;  
	private Duration interval = FIVE_HUNDRED_MILLIS;   
因此，如果不设置`withTimeout`、`pollingEvery`则相当于等待了500ms,并且请求了一次，要使用`FluentWait`应该依据实际需要进行设置。那有没有更好的方法呢，有的，请往下看。  

	WebDriverWait extends FluentWait<WebDriver>
	
	Wait<WebDriver> waiter = new WebDriverWait(driver, 10);
	WebElement element = waiter.until(new Function<WebDriver, WebElement>() {
			public WebElement apply(WebDriver driver) {
				return driver.findElement(By.id("element"));
			}
			
		});
	element.click();//something to do
`WebDriverWait`是继承于`FluentWait`的，并且实现了对功能进行了增强。从源码看出`WebDriverWait`的构造函数进行了如下的设置：

	withTimeout(timeOutInSeconds, TimeUnit.SECONDS);
    pollingEvery(sleepTimeOut, TimeUnit.MILLISECONDS);
    ignoring(NotFoundException.class);
设置了超时时间、每次请求的间隔为`sleepTimeOut`（默认500ms）、忽略了`NotFoundException`。因此直接使用`WebDriverWait`更加的省事。
###总结###
WebDriver提供了很多等待机制来增加selenium自动化的稳定性，只要合理利用是可以达到理想的效果的。   
无疑第一种方法sleep是最不可取的，是万不得已才用的一份方法，因为元素的加载与网络速度等客观因素直接相关。这个sleep的值是很难取的，值小了不行，值大了会造成case的运行速度缓慢。   
第二种方法是从全局的角度来解决元素查找问题的，在解决通用性的问题上有一定的优势，可以考虑使用。   
第三种方法是最好的，也是**推荐**的一种等待方式，很好的解决了动态元素的查找问题。   

---