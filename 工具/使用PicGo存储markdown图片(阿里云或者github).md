#### PicGo代替极简图床
之前使用极简床图，但是后来好像挂了，真是一件悲伤的事，最近才发现了一个神器，开源的`PicGo`，已经有各个平台的版本了。链接如下：https://github.com/Molunerfinn/PicGo/releases 去下载自己的平台即可。虽然你要是`Mac`，有`iPic`也是很ok的。

下载好之后怎么配置呢？

#### 配置阿里云
之前是注册阿里云认证之后就送一个`oss`存储（免费的），不知道现在还有没有，如果有的话就可以按照这个步骤配置下去。https://homenew.console.aliyun.com/

![](https://img-blog.csdnimg.cn/img_convert/27958f9e224d9edc6f5029668a201799.png)

打开`oss`之后，需要自己建立`Bucket`（存储空间名），存储区域（比如`oss-cn-qingdao`），除此之外还需要`keyid`和`keysecret`。（一定要打开公共读的权限，要不是访问不了的,但是不要打开公共写！！！）

![](https://img-blog.csdnimg.cn/img_convert/75e3fc873828c7069d92bd164284ded2.png)

点击自己的头像，选择`accesskeys`，新建一个`key`就可以了，我是新建了公共的。

![](https://img-blog.csdnimg.cn/img_convert/66f855c24c8472b4a51d7b872ad0bf5e.png)

安装`PicGo`之后，打开，选择阿里云，把上面的东西配置进去就可以了。

![](https://img-blog.csdnimg.cn/img_convert/785414926c1b0662e8607aa195f6905a.png)

然后选择上传区，就可以上传了，选择不同的`markdown`之类的，也可以选择剪贴板上传，上传成功之后就会有提示，连接就在粘贴板了，直接去粘贴就可以了。

#### 配置github
新建一个仓库，然后记得仓库的名字，点击头像，选择`settings`，`developer` `settings`--》`personal access token`--》新建一个`token`，记得勾选上权限，我全部选了，那个`token`只能查看一次，最好保存一下！！！

然后打开`PicGo`，选择`Github`图床，设置好就可以了。

![](https://img-blog.csdnimg.cn/img_convert/7668a762ad1413ab0f9700b7f0978bc9.png)

**此文章仅代表自己（本菜鸟）学习积累记录，或者学习笔记，如有侵权，请联系作者删除。人无完人，文章也一样，文笔稚嫩，在下不才，勿喷，如果有错误之处，还望指出，感激不尽~**

**技术之路不在一时，山高水长，纵使缓慢，驰而不息。**

**公众号：秦怀杂货店**

![](https://img-blog.csdnimg.cn/img_convert/7d98fb66172951a2f1266498e004e830.png)