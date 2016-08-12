---
layout: post
title: "Selenium-Grid2 配置"
date: 2013-01-10 19:37
comments: true
categories: selenium
tags: [ grid, selenium ]
---
###为什么要使用Selenium Grid ?###
* 分布式运行大规模的Test
* 能够通过一个中央点，很容易的运行不同OS上的不同browser
* 最小化对Grid的维护时间，并能充分利用虚拟设备

###Selenium Grid 部署与启动###
Hub :总控节点，连接调用Node。   
Node: 负责执行Tests,调用浏览器。 
<!--more-->
下面以selenium-server-standalone-2.27.0.jar版本为例：   
使用这样3台机器：   
  
* 10.81.14.170    
* 10.81.14.180     
* 10.81.14.190    

启动Hub（10.81.14.180）:

	java  -jar  selenium-server-standalone-2.27.0.jar -role hub 
在浏览器内打开：<http://10.81.14.180:4444/grid/console>可以查看Hub状态。也就是说Grid默认启动端口是4444，如果想切换为其他端口，则加`-port`参数。比如要切换为8888：

	java  -jar  selenium-server-standalone-2.27.0.jar -role hub  -port 8888

启动Node（10.81.14.170）: 

	java -jar selenium-server-standalone-2.27.0.jar -role node -hub http://10.81.14.180:8888/grid/register
同样的，也可以使用`-port`切换node端口，默认端口是5555.    
此处的node节点，也可以作为一个单机的远程节点存在，并同时支持RC,WebDriver。浏览器输入<http://10.81.14.180:8877/wd/hub>可以看到session信息。

然后，同样的启动10.81.14.180、10.81.14.190上的Node节点。

打开浏览器<http://10.81.14.180:8888/grid/console>,可以看到如下的界面：   

![](/images/blog/selenium-grid-console.png)    
 
至此，Selenium Grid2已经配置成功了。

###使用Grid运行Tests###
Selenium Grid2是向后兼容的，同时支持RC,WebDriver。
如果使用RC,即Selenium1，使用以下的方法：   

	Selenium selenium = new DefaultSelenium(“10.81.14.180”, 8888, “*firefox”, “http://www.baidu.com”);
使用WebDriver的话，使用以下的方法：   

	DesiredCapabilities capability = DesiredCapabilities.firefox();
	WebDriver driver = new RemoteWebDriver(new URL("http://10.81.14.180:8888/wd/hub"), capability);

可以看出所有的请求都发给了Hub,然后由Hub分配给匹配的节点来执行。   
那么，Hub是如何来分配的呢？往下看   
###Node配置###
默认，Node会启动11个浏览器实例:5 Firefox,5 Chrome, 1 Internet Explorer. 从Grid Console界面看出来，为什么每个机器上有22个实例呢？是这样的，Node为了同时支持RC与WebDriver两种协议，所以就是2＊11了。把鼠标放到各个浏览器图标上，就可以看出里面的配置区别了。  
内容类似：   

	{
          "browserName": "*firefox",
          "maxInstances": 5,
          "seleniumProtocol": "Selenium"
        }
或者

	 {
          "browserName": "firefox",
          "maxInstances": 5,
          "seleniumProtocol": "WebDriver"
        }
其中，seleniumProtocol就是定义的不同协议了。

如何修改Driver配置呢？可以从启动参数里操作。   

	-browser browserName=firefox,version=3.6,maxInstances=5,platform=LINUX

那Node默认启动的配置是什么呢？    
由于如果从启动参数里，配置这个多东西，很难写的。因此，官方很人性化的提供了JSON文件来配置。也就是说默认启动的配置如下：   
<http://code.google.com/p/selenium/source/browse/trunk/java/server/src/org/openqa/grid/common/defaults/DefaultNode.json>   
 
<http://code.google.com/p/selenium/source/browse/trunk/java/server/src/org/openqa/grid/common/defaults/DefaultHub.json>    

如果想自定义配置，直接对json文件修改，启动时，指定配置文件就可以了。   

	java -jar selenium-server-standalone.jar -role hub -hubConfig hubconfig.json 

仅仅就这样就行了？从博文<http://www.shenyanchao.cn/blog/2012/10/12/selenium-multiple-browser-support/>知道，浏览器的启动是要制定一些driver位置的，否则Node不知道怎么启动浏览器实例。因此需要进行指定：  

	java -jar selenium-server-standalone-2.27.0.jar -port 8877 -role node -hub http://10.81.14.180:8888/grid/register  -nodeConfig nodeconfig.json -Dwebdriver.chrome.driver="E:/selenium/chromedriver.exe" -Dwebdriver.ie.driver="E:/selenium/IEDriverServer.exe"


参考文档：  
<http://code.google.com/p/selenium/wiki/Grid2>
