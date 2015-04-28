---
layout: post
title: "PageObjects 设计模式"
date: 2012-10-16 16:40
comments: true
categories: selenium
tags: [ webdriver, selenium, PageObjects, PageFactory ]
---
##什么是Page Objects(翻译为：页面对象？)...##
简单的说，Page Objects是指UI界面上用于与用户进行交互的对象。它可以指整个页面，也可以指Page上的某个区域。Page Objects是你的test code的交互对象，是对实际UI的一种抽象模型化。通过Page Objects可以减少重复代码的编写，例如，很多页面都有同样的header，footer，navigator等部分，如果对这些进行抽象，只写一次就可以在其他地方通用了。  

**注意PageObjects与Page Objects是不一样的**，PageObjects用于特指采用Page Objects进行封装的一种设计模式（Design Pattern）,而不仅仅是多一个空格的区别。哈。
##如何实现PageObjects设计模式？##
一般情况下，对于一个Page Objects对象，它有两个方面的特征：  
 
* 自身元素(WebElement)  
* 实现功能 (Services)  
<!--more-->  

自身元素很好理解，就是实实在在的页面元素。而Page Object通常也都是实现一定的功能的。就Test的开发人员来说，更关心的是Page Objects它们实现了什么交互功能，而不是其内部的实现，因此，这里的功能与开发人员理解的功能是**不一样的**。   

以用户登录为例：在登录界面，点击登录后要么成功，转向首页。要么失败，出现提示出错信息。   
相信这是一个很容易理解的场景吧！  
Java Code可能类似如下：   

	public class LoginPage {
		//用户名录入框
		private WebElement usernameBox;
		//密码录入框
		private WebElement passwordBox;
		//提交按钮
		private WebElement submitButton;

    	public HomePage loginAs(String username, String password) {
			usernameBox.sendKeys(username);
			passwordBox.sendKeys(password);
			submitButton.submit();
        	return new HomePage(...)
    	}
    
    	public LoginPage loginAsExpectingError(String username, String password) {
       		 //  出错的username,password 仍留在LoginPage
   		 }
    
    	public String getErrorMessage() {
        // 获取错误信息
    	}
	}   
从上面可以看出，同时封装了元素以及功能。此处样例，元素是没有初始化的。可以通过类似于`driver.findElement()`的函数来直接进行初始化，另外WebDriver提供了一个PageFactory用于对PageObjects设计模式进行支持，下面将会讲到。  
通过上面的这段代码，也展现出了一个重要的问题，那就是assertion不应该在Page Objects内部，而应该由tests进行处理。Page Objects只是返回需要验证的信息即可。

##总结##
* public方法代表Page提供的功能
* 尽量不要暴露Page的内部细节
* 不要assertion
* 方法可以返回其他Page Objects
* Page Objects不用代表整个页面，可以是任意一个部分
* 一样的操作，不同的结果应该分开（正确登录，错误登录）   

##样例##
	public class LoginPage {
		private final WebDriver driver;
		// 用户名录入框
		private WebElement usernameBox;
		// 密码录入框
		private WebElement passwordBox;
		// 提交按钮
		private WebElement submitButton;

		public LoginPage(WebDriver driver) {
			this.driver = driver;
			if (!"Login".equals(driver.getTitle())) {
				throw new IllegalStateException("This is not the login page");
			}
			this.usernameBox = driver.findElement(By.id("username"));
			this.passwordBox = driver.findElement(By.id("passwd"));
			this.submitButton = driver.findElement(By.id("login"));
		}

		public HomePage loginAs(String username, String password) {
			usernameBox.sendKeys(username);
			passwordBox.sendKeys(password);
			submitButton.submit();
			return new HomePage(driver);
		}
	}

##PageFactory##
从上面的样例中，有没有发现每个元素都要进行`driver.findElement()`这样的操作，写起来好累啊，一堆重复性的代码。有没有更好的，更优雅的处理方法呢？**`org.openqa.selenium.support.PageFactory`**就是用来负责处理这个的，真Happy!   
下面以[百度搜索](http://www.baidu.com)作为样例场景，搜索一个关键字：	

	import org.openqa.selenium.WebDriver;
	import org.openqa.selenium.WebElement;
	import org.openqa.selenium.htmlunit.HtmlUnitDriver;
	import org.openqa.selenium.support.PageFactory;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;

	/**
 	* @author shenyanchao
 	* 
	 */
	public class BaiduSearchPage {
		public static final Logger LOG = LoggerFactory
			.getLogger(BaiduSearchPage.class);
		private WebElement wd;

		public void searchFor(String keyword) {
			wd.sendKeys(keyword);
			wd.submit();
		}

		public static void main(String[] args) {
			WebDriver driver = new HtmlUnitDriver();
			driver.get("http://www.baidu.com");
			BaiduSearchPage baiduPage = PageFactory.initElements(driver,
				BaiduSearchPage.class);
			LOG.info("before search url is:{}",driver.getCurrentUrl());
			baiduPage.searchFor("blueshen");
			LOG.info("after search url is:{}",driver.getCurrentUrl());
		}
	}
运行以上代码，发现已经可以正常运行，结果如下：

	......
	before search url is:http://www.baidu.com/
	......
	after search url is:http://www.baidu.com/s?wd=blueshen&rsv_bp=0&rsv_spt=3
可见，搜索后，已经转向了正确的搜索结果页面。然而WebElement是如何初始化的呢？玄机就在`BaiduSearchPage baiduPage = PageFactory.initElements(driver,BaiduSearchPage.class);`这行代码。PageFactory负责初始化了Page里的元素，amazing，用起来就是这么的优雅。   
那么下来，我就要问了：PageFactory是怎么定位元素的呢？   
>原来PageFactory初始化元素有一个惯例，样例中将WebElement的名称定为wd,那么PageFactory将按类似以下的形式对其进行初始化：    
`driver.findElement(By.id("wd"));`  
PageFactory认为wd是HTML元素的id或者name字段的值,并且优先从id开始查找。至此，我们终于知道怎么回事了。   

随着项目的变大，以及使用的更加深入，HTML元素的id，name字段并不一定唯一，并且Java Class的属性看起来都是一堆无意义的名称。这些要求我们必须要进行改进。幸好PageFactory已经提前考虑到了这一切，它支持annotations来显式定位元素。那么上述的百度搜索样例，可以修改为如下形式：   

	public class BaiduSearchPage {
		public static final Logger LOG = LoggerFactory
			.getLogger(BaiduSearchPage.class);
		@FindBy(how = How.NAME, using = "wd")
		@CacheLookup
		private WebElement serachBox;

		public void searchFor(String keyword) {
			serachBox.sendKeys(keyword);
			serachBox.submit();
		}
	......
	}
明确的指定HOW.NAME,using="wd",意为查找name="wd"的元素，并将其初始化赋值给searchBox这一有意义的属性名。其中@CacheLookup用于标识其只初始化一次，然后缓存起来备用。   

感觉还不够简洁吗？继续修改：  

 	@FindBy(name = "wd")
  	private WebElement searchBox;
这是其简略模式，还支持各种定位方式。   

		@FindBy(id="...")
		@FindBy(className="...")
		@FindBy(name="...")
		@FindBy(xpath="...")
		@FindBy(linkText="...")
		@FindBy(partialLinkText="...")
		@FindBy(tagName="...")
		@FindBy(css="...")
同时支持`@FindBys`用于支持列表元素查找定位，返回`List<WebElement>`类型。

**总之，利用PageObjects设计模式并且配合PageFactory使用，将使你的自动化测试优雅、易懂、易维护。**