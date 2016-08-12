---
layout: post
title: "提交代码到GitHub SSH错误解决方案"
date: 2013-09-16 18:08
comments: true
categories: git
tags: [ git, ssh, connection ]
---
最近，由于公司内部的网络改造，在`git push`的时候提示如下信息：

    ssh: connect to host github.com port 22: Connection timed out
    fatal: Could not read from remote repository.
    
    Please make sure you have the correct access rights
    and the repository exists.
看字面的意思就是连接超时了，什么原因造成的呢，这个估计是公司网络禁用了SSH的22端口导致的。所幸git提供了https、git、ssh三种协议来读写。

运行`git config --local -e`打开配置信息。   
修改其中的    

    url = git@github.com:username/repo.git
为   

    url = https://username@github.com/username/repo.git
这样就改为使用https协议了。提交什么的就OK了。

后来，我再使用Octopress提交博客的时候问题又出现了。也就是在`rake deploy`的时候，又走ssh了。此时只要进入_deploy目录，把上述步骤再执行一次就好了。就是每次提交都要输入用户名密码而已。  
