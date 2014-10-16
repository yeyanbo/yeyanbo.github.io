---
layout: post
title: 使用 Apache FOP 生成 PDF 文档研究记录
tags: [fop, pdf, rtf, java]
---

如何为 PDF 文档添加索引目录树
---------------------------------
中间文件*.fo的格式如下生成：

````xml
<fo:bookmark-tree>
    	<fo:bookmark internal-destination="sec0">
       		<fo:bookmark-title>Adding Fonts to FOP</fo:bookmark-title>
    		</fo:bookmark>
    		<fo:bookmark internal-destination="sec1">
        		<fo:bookmark-title>Adding additional Type 1 fonts</fo:bookmark-title>
        		<fo:bookmark internal-destination="sec1-1">
            			<fo:bookmark-title>Generating a font metrics file</fo:bookmark-title>
        		</fo:bookmark>
        		<fo:bookmark internal-destination="sec1-2">
            			<fo:bookmark-title>Register the fonts within FOP</fo:bookmark-title>
        		</fo:bookmark>
    	</fo:bookmark>
</fo:bookmark-tree>
````

其中``internal-destination="sec0"`` 为目录与正文链接的锚点 id 定义。

如何强制换页
--------------------------------

输出PDF文档时，用``<fo:block  break-after="page"/>``命令可以正常换页，而输出 RTF 时，不起作用。

如何支持中文
-------------------------------------

先要把中文字体给 FOP 指出来：

//TODO

````
fop -c conf/userconfig.xml -xml historycreator.com/books.xml -xsl historycreator.com/books.xsl -pdf historycreator.com/books.pdf
````

注意这里使用了参数-c，指定使用我们自己写的userconfig.xml为配置文件。
执行结果：在historycreator.com文件夹下生成了一个books.pdf文件,并且可以正确显示中文.