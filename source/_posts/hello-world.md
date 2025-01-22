---
title: 搭建博客
lang: zh-CN
---
虽然遇到了一些问题，不过都很快找到方法解决了。看来之前安装VS和openCV的坑不是白踩的。

Maxw终于有了自己的博客啦！有点激动，也有点犹豫跟纠结，虽然理由已经充分到说服自己无数遍，但我仍无法预测自己毅然走出专业的圈走进写代码的坑是不是更加正确的选择...

## 2/23-2/24

### git初试

follow [枫叶的教程](https://zhuanlan.zhihu.com/p/103391101)
problem occur: 
can' t clone existing repostory with git bash reports timeout 
or OpenSSL SSL_read: Connection was reset, errno 10054

从GitHub上clone仓库的时候，试了几次，有时git bash报错10054，有时timeout。

我先是按[网上教程](https://blog.csdn.net/weixin_44795128/article/details/119214048?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7EHighlightScore-1.queryctrv2&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7EHighlightScore-1.queryctrv2&utm_relevant_index=1)改了git的global config，还是报错

想到自己上GitHub都要科学方法，就觉得应该是代理的问题。git应该并非使用系统代理设置，而是需要自行设置，果不其然。

solve: 
setup proxy for git, following [git设置代理](https://www.jianshu.com/p/290152303598)

### 博客搭建

follow [枫叶的教程合集](https://www.zhihu.com/column/c_1201860091307458560)

感谢枫叶大佬的教程，有时间一定上知乎评论区里发个反馈。

安装node.js与插件的过程中报错属于node.js本地的文件夹无法访问。一开始觉得很奇怪，明明特意从c盘卸载了安别的盘，后来我想起来那个盘是我从c盘分出去的了，草（感叹词）。所以应该是因为也需要管理员权限（也可能是我把git安到c盘了？）。果然管理员启动bash之后，安装就顺畅了。

枫叶的教程里面设置npm的环境变量那里可能有点问题，设置好之后应该是不需要像网上一些教程说的那样安装两遍npm的。npm所在目录需要保留而不是更改，再按教程添加node的目录。在系统变量里添加node的路径后，path里需要添加对应的%NODE_PATH%，这也是教程里未提及的。

其他问题应该就只是跟版本有关了，比如GitHub把主分支从master改成main，创建个人网页的setting单独分页了等等。以及博客更新后无需删除.git重新上传，hexo的操作估计是有延迟的。

我跳过了教程里设置自己的域名的部分。（以后博客里内容多了再为它花钱吧）

在菜单中增加新页面需要同时hexo new page以及在主题的config文件中设置（参考枫叶的教程）

### 一些感叹

虽然我对于文件系统和权限的理解仅限一鳞半爪，但在配置网站的过程中，这些知识还是帮了我不少忙。不愧是程序员的基本功。

### 一些bug

git报错the remote end hung up unexpectedly：照此教程解决了：
https://jingyan.baidu.com/article/afd8f4de38d87174e386e967.html

### 一些bug 2.0

。。。又给git的proxy摆了一道没办法deploy博客，不想用的时候一定要记得`git config --global --unset http.proxy `重置啊！设置回来则是`git config --global http.proxy proxyaddress:port`

最近总感觉自己记性有点差，这是写完博客后如何deploy:
在博客的根文件夹（那个有.deploy_git的文件夹）打开git bash，输入：
```
hexo clean
hexo generate
hexo deploy
```