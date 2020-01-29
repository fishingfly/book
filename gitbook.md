# gitbook

gitbook 首先是一个软件，正如上面定义的那样，它使用 Git 和 Markdown 来编排书本，如果用户没有听过 Git 和 Markdown，那么 gitbook 可能不适合你！

一般有两个方式使用gitbook，最终都会呈现给用户，两种方式分别为：gitbook本地制作和gitbook页面制作

### gitbook本地制作

原理：gitbook本地使用+推送到github+设置github page=实现和gitbook一样的显示效果，下面是步骤：

安装Node.js，这个你直接上网百度吧，安装教程很多。

安装gitbook：npm install -g gitbook-cli

检查gitbook是否安装成功：gitbook -V

开始使用gitbook gitbook init

会出现两个文件README.md和SUMMARY.md

在SUMMARY.md文件内容中写入如下内容：

```text
* [前言](README.md)
* [第一章](Chapter1/README.md)
  * [第1节：衣](Chapter1/衣.md)
  * [第2节：食](Chapter1/食.md)
  * [第3节：住](Chapter1/住.md)
  * [第4节：行](Chapter1/行.md)
* [第二章](Chapter2/README.md)
* [第三章](Chapter3/README.md)
* [第四章](Chapter4/README.md)
```

再继续一次`gitbook init`，这时可以看到，gitbook就自动生成了这些目录了（第一章，第二章等目录结构）。再运行`gitbook serve`来预览这本书籍

本地浏览器访问[http://localhost:4000](http://localhost:4000)。就可以访问到你的gitbook了。但到这步为止仅限你本地使用，下面需要进行build操作和Github配置

使用`gitbook build`命令来生成最终的项目：

```text
~ gitbook build
​
info: 7 plugins are installed 
info: 6 explicitly listed 
info: loading plugin "highlight"... OK 
info: loading plugin "search"... OK 
info: loading plugin "lunr"... OK 
info: loading plugin "sharing"... OK 
info: loading plugin "fontsettings"... OK 
info: loading plugin "theme-default"... OK 
info: found 5 pages 
info: found 0 asset files 
info: >> generation finished with success in 1.0s !
```

命令执行结束后，会在项目下生成`_book`的文件夹,此文件夹就是最终生成的项目。 在`_book`文件夹里有一个`index.html`文件，这个文件就是文档网站的HTML入口，把`_book`文件夹复制到服务器，然后把web服务的入口引向`index.html`即可完成文档网站的部署。（你可以自己找个tomcat试下，应该可以的），不过我这里不是使用tomcat来部署，而是使用github page来部署。

（如果你想查看输出目录详细的记录，可使用`gitbook build ./ --log=debug --debug`来构建查看。）

去Github上创建一个公开仓库，取名为随意（我取名为test）。创建完后，你到本地\_book目录下执行：

```text
git init
git add -A
git commit -m "for deploy"
git push -f github仓库地址 master:gh-pages
```

你需要强行覆盖远程仓库内容，将本地分支推送到远程的gh-pages分支，为啥是ph-pages分支，你可以去搜一下github page的使用就知道了。推送上去后，你去github上的仓库的settings中，下拉,找到Github page。可以看到一个地址，我的是[https://fishingfly.github.io/test/](https://fishingfly.github.io/test/)。然后你访问后，就可以看到你自己的gitbook了。是公开的，所有人都能看到。

### gitbook线上制作

直接去[https://www.gitbook.com/](https://www.gitbook.com/) 官网注册一个账号，创建一个公开空间就可以了，其实操作比较简单，自己去探索下，最后在share中查看地址，这个地址就是可以让其他人查看你的gitbook的地址，官网是都是通过页面操作来编写文章的，很简单。不在这里详述。

我对于Gitbook的简单使用我就探索到这了，我的基本要求已经满足了，

参考文章：[https://segmentfault.com/a/1190000017960359](https://segmentfault.com/a/1190000017960359)

