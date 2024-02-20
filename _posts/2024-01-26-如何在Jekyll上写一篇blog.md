---
layout:     post   				    # 使用的布局（不需要改）
title:      如何在Jekyll上写一篇blog 				# 标题 
subtitle:   边学，边记录       #副标题
date:       2024-01-26 				  # 时间
author:     Tong 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 教程
---

## 如何在Jekyll上写一篇blog
>文件名格式：每一篇文章文件命名采用的是`2024-01-26-Hello-World.md`时间+标题的形式，空格用`-`替换连接。



在博客前言部分输入blog信息

````yaml
layout:     post   				    # 使用的布局（不需要改）
title:      My First Blog 				# 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2024-01-26 				# 时间
author:     Tong 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 生活
typora-copy-images-to: ../assets/images #复制图片存储路径
typora-root-url: ../../blog  #博客根路径
````



编写正文内容



## 上传Blog到Github

当我们保存编辑好的blog文件到_posts目录下后，我们要做的是把文件提交到Github上，让GitHub pages进行静态渲染

**添加修改后的文件**

> git add . 

提交

> git commit -m "first commit"

把远程仓库最新的拉取到本地

> git pull 

如果有冲突就合并

> git pull --no-rebase

推送到远程仓库

> git push



大功告成，等几分钟GitHub pages就会自动部署完成啦







以后慢慢揭晓。。。

哈哈哈哈哈