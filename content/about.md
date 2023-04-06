---

title: 'About'
date: 2022-10-23T07:51:20+08:00
tag:
  - mdbook
  - makedown
  - hugo
---


> 尝试迁移到了 hugo

数年来 前前后后，经历了 asp+access > wordpress > typecho > wordpress > cdsn > typecho > wordpres > vuepress > gitbook > mdbook > hugo

也丢失了不少数据。。。


部署在 github 依赖 actions 自动从私有库生产后 push 到 开启了page的公开库


wordpress 对多域名的支持很懒，需要大量的修改和插件处理。

typecho sqlite 有一些莫名其妙的小错误，

并且都基于php的 需要服务器单独处理，不适合只是文章功能的简单站点


hexo vuepress 都是基于nodejs的，本地调试都需要安装nodejs 麻烦

mdbook 简单易用，但是 需要手动编写列表，另外搜索功能对中文支持很差  多语言支持需要手动处理 列表文件


hugo 相对mdbook 繁琐了很多，但是功能也好很多，而且自带的搜索 功能好用，不需要第三方，模板也比较丰富。


目前基于 github 私有库 actions 自动push 到公开库，然后国内走cdn 境外直链github
