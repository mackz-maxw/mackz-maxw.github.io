---
title: 摸鱼时间
---

心血来潮想找几个javascript小游戏玩玩，github上找到了untrusted这个游戏。

## 3/18

想打开网页直接玩来着，结果网页显示一些资源无法访问。于是按issues里的问题换了源，还是没办法。看来只好搭个本地服务器了。

直接down zip文件到了本地（文件有点大 怕git拉到本地中途出问题），git bash安装了http-server，http-server可以这么启动：

 `http-server [path]`

结果跟网页端一样无法显示...一看报错发现没有编译.js文件出来，这就网上搜一圈看看怎么弄。

下载git bash的时候就看到附带了mingw，结果不能直接用make命令，而是得用mingw32-make，好家伙，然而整完还报错：

`Makefile:14: *** File mods/default/intro.js not found!.  Stop.`

本来以为是下载的时候出了问题，上github一看，人家的描述是会自动生成这个文件，那我这是哪里出了问题？

整了挺久累了，下次再更🕊~

对了！make不能用cmd，会报奇怪的错！git bash就不会。

## 3/18

在github的issues里找到了这个解决换源，虽然说的是网页问题，但也可以改下载下来的code解决404问题。

> Here's a solution:
>
> 1. Right click and click “Save As...”
> 2. Edit the html file that just saved, overwrite jQuery reference `<script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>` as `<script src="http://libs.baidu.com/jquery/1.11.3/jquery.min.js"></script>` or other reference you trust.
> 3. run the html.
>

最后就是上面的file缺失问题了，我找到了一个较早的branch，copy了里面untrusted.js的代码并整到本地的对应文件夹

`E:\works\untrusted-master\scripts\build`

这样就不用make了，虽然感觉可能会出些错，但是我只是想玩个游戏，先这样啦~

对了，别忘了

`http-server *:/.../untrusted-master`

（*：code所在盘符，...：code所在目录）

终于可以玩了！



这个游戏里用setPhoneCallback蛮多次的，在这里存下这些代码，免得往回翻了。

```
 map.getPlayer().setPhoneCallback(function () {
        var player = map.getPlayer();
        if(player.getColor() == "#0f0"){
        	return player.setColor("#f00");
        }
        else if(player.getColor() == "#f00"){
        	return player.setColor("#ff0");
        }
        else if(player.getColor() == "#ff0"){
        	return player.setColor("#0f0");
        }

    });
```

