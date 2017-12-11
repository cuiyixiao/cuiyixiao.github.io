# github jekyll配置blog
##  一。安装相关软件
- ruby	sudo apt-get install ruby
- ruby-dev	sudo apt-get install ruby-dev
- jekyll	sudo gem install jekyll
- bundler	sudo gem install bundler
- nodejs	sudo apt-get install nodejs
- 对于其他gems	可以尝试bundle update更新gems包
## 二。github账号
- 有一个自己的github账号
- 建一个库名字为username.github.io（username为自己github账号的名字）
## 三。向远程库推送主题文件
 -  git clone https://github.com/username/username.github.io  (username位自己的github账号名字)，这步是为了将远程空的库先克隆到本地。
- setting - github page - choose theme 在里面选择自己喜欢的主题，下载并解压到上一步克隆的目录里
- git add --all
- git commit -m "Initial commit"
~$git push -u origin master （如果未填写ssh需要输入github的用户名和密码）

## 四。改写index.md显示自己的博文
- https://www.jekyll.com.cn/docs/posts/中有相关说明，yaml头信息需要保留，下面的直接改成链接里给的一段代码即可。
- 建立文件夹_posts，就可以在里面开始写博文啦，可以用markdown或者Textile来写
- 打开https://username.github.io/就可以看到自己的博客了

# ps ：感谢前辈的帮助，a li ga do

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1512990414548&di=ce8b9fa2c9300726ded85765a966d1c0&imgtype=0&src=http%3A%2F%2Fi2.hdslb.com%2Fbfs%2Fface%2F38a807a805787f8d049ae5617117068635133f74.jpg)

- 参考的blog   https://github.com/Irving-cl/Irving-cl.github.io 作者公司的一位前辈
- 参考的文档1 https://pages.github.com (英文文档很关键，git官方文档)
- 参考的文档2 http://wowubuntu.com/markdown/#link（markdown相关语法）
- 参考的文档3 https://www.jekyll.com.cn/docs/posts/ （jekyll文档）
- 参考的文档5 https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/（jeky with blog）
### 听说经常撸管有三个毛病，1，话说不全。2，数总数查的不对。3，
