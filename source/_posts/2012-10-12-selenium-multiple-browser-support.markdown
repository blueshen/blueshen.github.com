---
layout: post
title: "Selenium WebDriver的多浏览器支持"
date: 2012-10-12 14:56
comments: true
categories: selenium
tags: [ webdriver, selenium, Firefox, Chrome, Driver ]
---
##Selenium WebDriver支持哪儿些浏览器？
按照官方文档的说法，现阶段有以下的drivers: 
>* ChromeDriver  
>* InternetExplorerDriver   
>* FirefoxDriver   
>* OperaDriver   
>* HtmlUnitDriver   
>* AndroidDriver(mobile testing)  
>* OperaMobileDriver(mobile testing)  
>* IPhoneDriver(mobile testing)   
<!--more-->   

##为什么selenium自动化case在一个浏览器运行的很好，换为另外一个浏览器则不行？   
###一个Driver可以打开浏览器，另外一个Driver却不行？   
WebDriver是通过调用native浏览器来操作的，浏览器之间的差异注定会出现一些问题。下面以InternetExplorer,Firefox,Chrome为例进行说明：  
####InternetExplorer：  
> 1.它分不同的版本，版本之间差异很大。InternetExplorerDriver支持IE6、7、8、9。操作系统支持XP、Vista、Windows 7。   
>2.InternetExplorerDriver同时支持32/64bit的浏览器，这个取决于你用的是什么版本的[IEDriverServer.exe](http://code.google.com/p/selenium/downloads/list)。   
>3.要求条件如下：
>>* [IEDriverServer](http://code.google.com/p/selenium/downloads/list)在系统环境的PATH内（selenium2.26.0+版本推荐方式）。或者设置`webdriver.ie.driver`系统属性。   
	 `System.setProperty("webdriver.ie.driver", "D:\\IEDriverServer.exe");`  
	` WebDriver driver = new InternetExplorerDriver();`
>>* 在windows vista、windows7操作系统中，如果使用IE7+的浏览器，应该保证浏览器的**保护模式**都处于**同一状态**[开启或者关闭]。如果不一致，那么报错信息类似于`Caused by: org.openqa.selenium.WebDriverException: Unexpected error launching Internet Explorer. Protected Mode settings are not the same for all zones. Enable Protected Mode must be set to the same value (enabled or disabled) for all zones. (WARNING: The server did not provide any stacktrace information)`   
不会设置吗？   
操作如下：打开浏览器->Internet选项 ->安全->启用保护模式。保证Internet、本地Intranet、受信任的站点、受限制的站点4个zone保护模式一致就OK   
>>* 为了确保能获得正确的坐标点，要把浏览器的缩放设为100%。   
设置方法：打开浏览器->页面->缩放(Z)->100%   

参考<http://code.google.com/p/selenium/wiki/InternetExplorerDriver>

#### Firefox:  
>1.Firefox不像InternetExplorer一样，用户可以自定义安装路径。因此使用时，需要制定firefox.exe的安装路径。  
怎么指定?
>>+ java code: `System.setProperty()`   
>>+ 命令行：`-DpropertyName='value'`   


>2.系统变量的值为：`webdriver.firefox.bin`，以及其他的key值，详见参考页面。webdriver.firefox.bin用来指定Firefox的安装路径。如不设置，默认从%PROGRAMFILES%\Mozilla Firefox\firefox.exe加载。**个人强烈建议，即使安装在默认路径也进行指定**。   
>3.Java代码如下：  
	System.setProperty("webdriver.firefox.bin", "C://Mozila/firefox.exe"); 	
    WebDriver driver = new FirefoxDriver();  
>其中firefox的安装路径，按情况自行替换。

参考<http://code.google.com/p/selenium/wiki/FirefoxDriver>   
####Chrome:   
chrome要求条件如下：
>1.Chrome应当安装在默认路径下（如果是从官方下载的，安装后直接都是默认路径）。
><table border=”1px">
<tbody>
<tr><td>OS</td><td>默认位置</td></tr>
<tr><td>Linux</td><td>/usr/bin/google-chrome</td></tr>
<tr><td>Mac</td><td>/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome</td></tr>
<tr><td>Windwos XP</td><td>%HOMEPATH%\Local Settings\Application Data\Google\Chrome\Application\chrome.exe</td></tr>
<tr><td>Windwos Vista</td><td>C:\Users\%USERNAME%\AppData\Local\Google\Chrome\Application\chrome.exe</td></tr>
</tbody>
</table>    
>2.需要下载相应版本的[chromedriver](http://code.google.com/p/chromedriver/downloads/list)，用来架起chrome浏览器与webdriver之间的桥梁。   
>3.与FirefoxDriver差不多，需要设置chromedriver的路径。key值为：webdriver.chrome.driver.   
>4.Java代码如下：   
	System.setProperty("webdriver.chrome.driver", "C://drivers/chromedriver.exe"); 	
    WebDriver driver = new ChromeDriver();    
需要注意的是，chrome浏览器会自动更新，而[chromedriver](http://code.google.com/p/chromedriver/downloads/list)也是不断更新的。如果chrome版本太新，而chromedriver没有相应的更换，会造成只是打开chrome浏览器，而不进行任何操作的问题。另外，ChromeDriver只适用于chrome 12.0.712.0+,如果需要使用更老的版本，见参考页面的详细描述。   
 
参考<http://code.google.com/p/selenium/wiki/ChromeDriver>   

###在一个浏览器里，元素可以找到或者可以操作，而在另外一个浏览器内则不行，为什么？
不同浏览器之间解析DOM以及响应事件的机制不同，难免会有一些不兼容性。解决方法：
>1.元素定位，通常是由于DOM解析不同造成的，可以使用不同的findElement方法进行实验，如id,class,xpath等。这个没有统一的结论，大多数情况下id是最靠谱的。**推荐！**   
>2.事件的响应，这个如果存在问题，一般比较难解决。通常是由于浏览器之间的差异造成的。可以通过使用selenium更高的版本，或者更换浏览器的版本来解决。或者想一下，有没有其他的方式，换个事件来绕过去，总有办法的。如果实在解决不了，那也只能暂时是这样了。