---
layout: post
title: 使用 Apache FOP 生成 PDF 文档研究记录
tags: [fop, pdf, java]
---

如何为 PDF 文档添加目录树
--------------------------------------------------
中间文件*.fo的格式如下生成：

```` xml
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
