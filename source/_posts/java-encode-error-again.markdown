---
layout: post
title: "又见Java乱码"
date: 2014-03-27 21:43
comments: true
categories: java
tags: [ dom4j, jenkins, java, 编码 ]
---
### dom4j解析xml

dom4j提供了一个`DocumentHelper`来解析xml内容，此处的内容是String类型的。下面是其源码：

```java
    public static Document parseText(String text) throws DocumentException {
        Document result = null;
        SAXReader reader = new SAXReader();
        String encoding = getEncoding(text);

        InputSource source = new InputSource(new StringReader(text));
        source.setEncoding(encoding);

        result = reader.read(source);

        // if the XML parser doesn't provide a way to retrieve the encoding,
        // specify it manually
        if (result.getXMLEncoding() == null) {
            result.setXMLEncoding(encoding);
        }

        return result;
    }

    private static String getEncoding(String text) {
        String result = null;

        String xml = text.trim();

        if (xml.startsWith("<?xml")) {
            int end = xml.indexOf("?>");
            String sub = xml.substring(0, end);
            StringTokenizer tokens = new StringTokenizer(sub, " =\"\'");

            while (tokens.hasMoreTokens()) {
                String token = tokens.nextToken();

                if ("encoding".equals(token)) {
                    if (tokens.hasMoreTokens()) {
                        result = tokens.nextToken();
                    }

                    break;
                }
            }
        }

        return result;
    }
```

从以上的代码中可以看出，解析过程中是使用XML的头`<?xml version="1.0" encoding="UTF-8"?>`来获取编码信息的。
<!--more-->

### 如何读取文件到String

```java
    public String loadXmlRule() {
        InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("file.xml");
        String xmlContent = null;
        try {
            xmlContent = IOUtils.toString(inputStream);
        } catch (IOException e) {
            LOG.error("read xml:{} io error", e);
        } finally {
            IOUtils.closeQuietly(inputStream);
        }
        return xmlContent;
    }
```

[getResourceAsStream将文件读为字节流](<http://stackoverflow.com/questions/5590451/getresourceasstream-what-encoding-is-it-read-as>)，不牵涉到字符编码问题。但是当你把这个inputStream转为String的时候，就需要指定字符编码了。否则不知道按什么编码规则解析字节流到字符。不知道什么编码的情况下，程序可能就会从系统变量取默认的字符编码，也就是LANG值。这个时候在LINUX，WINDOWS下表现的可能就不一致。因此必须显式的指明编码。

```java
xmlContent = IOUtils.toString(inputStream);
转换为：
xmlContent = IOUtils.toString(inputStream，"UTF-8);//假设文件是UTF-8
```

由此，一定要慎重使用编码。**永远不要相信默认编码**。

### Jenkins/Hudson中shell command的编码

遇到这样的情况，在jenkins的机器上直接执行shell命令，与在jenkins job中执行shell的默认编码是不一样的。机器上直接执行默认从环境变量里取的，但是jenkins job的编码是走的jenkins node上的默认编码配置。为了防止出现类似的问题，可以在jenkins job中提前指定特定的编码。
