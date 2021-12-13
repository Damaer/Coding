## 安装前提
确认电脑已经安装好 `node` 和 `npm` 环境。 如果还没有装好，那需要执行下面的步骤：
1.进入官网：https://nodejs.org/zh-cn/ ， 下载长期支持版。
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210106234542.png)

2.安装就直接下一步就可以了，默认会把环境变量添加进去。
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210106234734.png)

3.直到finish，打开cmd命令行，查看环境变量以及版本。（**此时你们看到的应该还是只把node.js的根目录添加到环境变量path**）
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210106235050.png)

4.运行命令修改npm的文件夹前缀和缓存目录,配置镜像站。
```shell
npm config set prefix "D:\nodejs\node_global"
npm config set cache "D:\nodejs\node_cache"
npm config set registry=http://registry.npm.taobao.org
```

然后使用`npm config list`就可以看到自己的配置：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210106235503.png)

还需要增加一个环境变量，是node的modules的环境变量(我的nodejs在D盘根目录下，你们的要自己根据实际情况)：
```shell
D:\nodejs\node_global\node_modules
```
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210106235744.png)

5.然后如果使用`npm`安装了东西，但是找不到该命令，则还需要在Path中，把我们node的全局文件夹添加进去环境变量中。

```shell
D:\nodejs\node_global
```

![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210107000041.png)

这样我们就可以愉快的安装东西了。


## docsify走起
官网：
https://docsify.js.org/#/

废话我就不多说了，直接安装`docsify-cli` :
```shell
npm i docsify-cli -g
```

然后我们建立一个测试文件夹叫`note`，命令行进入这个文件夹：
```shell
cd note
docsify init ./docs
```
就成功了！！！看到它叫你执行命令,本地启动一下：
```shell
docsify serve ./docs
```
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210107000543.png)

这样就可以在本地http://localhost:3000打开了，神奇~（修改内容后保存就可以，不需要重新启动）

![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210107000719.png)

## 美化一下
说实在话，挺丑的，那就美化一下：
先加一个封面，需要在`index.html中，把下面的属性设置为true
```shell
coverpage: true
```
然后新建一个文件`_coverpage.md`:
```txt
# Mybatis摸索之路


> 这是我自己的笔记啊啊啊啊

[CSDN](https://blog.csdn.net/Aphysia)
[滚动鼠标](#introduction)
```
然后它就变成这样了：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210107001308.png)

我们还需要一个侧边栏，再将侧边栏属性打开：
```shell
loadSidebar: true
```
然后新建一个侧边栏的文件`_sidebar.md`:
```txt
- Note

  - [第一章节](第一章节.md)
  - [第二章节](第二章节.md)
  - [第三章节](第三章节.md)
```
然后就变成这样了：
![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210107001644.png)

其中中间那部分使用的是`README.md`的内容，其他的index.html的内容如下(自己根据需要设置，如果有更高级的需求，建议去官网查文档！！！)
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>docsify-demo</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Description">
  <meta name="viewport"
    content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//unpkg.com/docsify/lib/themes/vue.css">
</head>

<body>
  <div id="app"></div>
  <!-- docsify-edit-on-github -->
  <script src="//unpkg.com/docsify-edit-on-github/index.js"></script>
  <!--Java代码高亮-->
  <script src="//unpkg.com/prismjs/components/prism-java.js"></script>
 <!--全文搜索,直接用官方提供的无法生效-->
 <script src="https://cdn.bootcss.com/docsify/4.5.9/plugins/search.min.js"></script>
 <!-- 复制代码到剪贴板 -->
  <script src="//unpkg.com/docsify-copy-code"></script>
  <!-- 图片缩放 -->
  <script src="//unpkg.com/docsify/lib/plugins/zoom-image.js"></script>
  <!-- 字数统计 -->
  <script src="//unpkg.com/docsify-count/dist/countable.js"></script> 
  <script>
    window.$docsify = {
      name: 'docsify-demo',
      repo: 'https://github.com/Damaer/Mybatis-Learning',
      maxLevel: 5,//最大支持渲染的标题层级
      subMaxLevel: 3,
      homepage: 'README.md',
      coverpage: true,
      loadSidebar: true,
      auto2top: true,//切换页面后是否自动跳转到页面顶部
       //全文搜索
       search: {
        maxAge: 86400000, // 过期时间，单位毫秒，默认一天
        paths: 'auto',
        placeholder: '搜索',
        noData: '找不到结果',
        // 搜索标题的最大程级, 1 - 6
        depth: 3,
      }
    }
  </script>
  <script src="//unpkg.com/docsify/lib/docsify.min.js"></script>
</body>

</html>
```

## 如何部署到github
下面讲讲如何部署，首先我们需要有一个远程的仓库，我默认你有了，使用命令初始化文件夹，关联远程仓库
```shell
git init
git remote add origin "自己在三方代码托管平台上所创建仓库对应的地址"
```

`push`代码到远程仓库就可以了，`git`的操作就不仔细讲了，或者自己把远程的仓库先`clone`下来，再用`docsify`创建文档，然后提交，也是ok的。

提交上去之后，我们需要做一个操作，在`settings`下有一个`GitHub Pages`,选择构建分支和文件目录即可。我使用的是`master`，根目录的`docs`文件夹。然后你就可以看到已经发布成功了，直接访问网址就可以。

PS：项目是我的其他项目地址，但是流程是一致的。

![](https://markdownpicture.oss-cn-qingdao.aliyuncs.com/20210107002821.png)


### 坑点
我打不开网址！！！是因为电信会屏蔽一些网站，也就是被qiang了，懂的都懂，如果要访问的话，可以修改DNS，或者搞一把梯_子。

