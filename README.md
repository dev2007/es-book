# 背景

> 为什么要写这个？

在使用Spring Boot开发项目过程中，需要对接使用elasticsearch，查阅资料得知，官方主推使用Java High Level REST Client。

但是各种博客资料对REST Client的介绍主要在于如何引入、配置，官方文档以用户是熟悉elasticsearch的基本API操作为前提来进行介绍，将API文档直接扔你脸上，让人看得一脸茫然，虽然看懂了每个字在说什么，连起来却不知道API要执行的意图。

因此，想在学习的过程中，试图翻译一下官方文档，并引入对elasticsearch的REST操作API的介绍，将REST Client的java代码与REST API对比介绍、学习。文章结构会与官方文档有差异，尽量按照我们习惯的学习过程展开，从一般的增、删、查、改，从简单操作到复杂组合，希望能在整个文章的整理过程中，加强对elasticsearch的学习与理解。

翻译水平有限，对elasticsearch的理解和说明又有基于个人的理解，一定存在差错和谬误，欢迎指正，也请分享您的经验。
