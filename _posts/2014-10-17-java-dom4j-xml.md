---
layout: post
title: 利用Dom4J进行XML操作
summary: 在进行Java编程时，很多时候都需要对XML文件进行操作，本文记录了在使用过程的一点用法
tags: [java dom4j xml xsd xslt]
---

## 如何进行XML的Schema校验



## 如何进行XML的XSLT转换


### 解决XSL文件的引入其他xsl文件的问题

在XSL文件中是可以引入其它的XSL文件的，例如：
``` xml
<xsl:import href="other_xsl_file.xsl"/>
```
如果不加处理，转换程序是无法定位到引入文件的：

``` java
/**
 * 解决xsl文件中xsl:import xsl:include导致的文件路径问题
 */
public class MyURIResolver implements URIResolver {

    @Override
    public Source resolve(String href, String base) throws TransformerException {
        try {
        	FileInputStream inputStream = new FileInputStream(AppConfigReader.getResourceUrl() + "xslt/" + href);
            return new StreamSource(inputStream);
        } catch (Exception ex) {
            ex.printStackTrace();
            return null;
        }
    }
}

/**
 * 然后在TransformerFactory工厂类中引用
 */
TransformerFactory factory = TransformerFactory.newInstance();
factory.setURIResolver(new MyURIResolver()); 
```

## XPath操作

使用Dom4j操作Xpath时，需要引入jaxen.jar