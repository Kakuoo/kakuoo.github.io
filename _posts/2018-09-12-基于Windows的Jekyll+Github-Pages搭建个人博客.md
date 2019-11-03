---
layout: post
title: A step-by-step guide to setting up Jekyll on Windows
subtitle: 突然想拥有一个属于自己的博客
gh-repo: Kakuoo/kakuoo.github.io
gh-badge: [star, fork, follow]
tags: [Jekyll]
comments: true
---


* This will become a table of contents (this text will be scraped).
{:toc}

# 前言——突然想有一个属于自己的博客

Github很好的将代码和社区联系在了一起，于是发生了很多有趣的事情，世界也因为他美好了一点点。Github作为现在最流行的代码仓库，已经得到很多大公司和项目的青睐，比如jQuery、Twitter等。为使项目更方便的被人理解，介绍页面少不了，甚至会需要完整的文档站，Github替你想到了这一点，他提供了Github Pages的服务，不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。

Github Pages 是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在 Github 上，我们可以选择使用 Github Pages 默认提供的域名 github.io 或者自定义域名来发布站点。Github Pages 支持自动利用 Jekyll 生成站点，也同样支持纯 HTML 文档，将你的 Jekyll 站 点托管在 Github Pages 上是一个不错的选择。

所以首先得在自己电脑上安装jekyll环境，这样就能在本地编辑预览自己的博客网站，弄好后直接上传到Github就可以发布自己的博客网站了。

Github Pages有以下几个优点：

- 轻量级的博客系统，没有麻烦的配置
- 使用标记语言，比如Markdown 无需自己搭建服务器
- 根据Github的限制，对应的每个站有300MB空间
- 可以绑定自己的域名

当然他也有缺点：

- 使用Jekyll模板系统，相当于静态页发布，适合博客，文档介绍等。
- 动态程序的部分相当局限，比如没有评论，不过还好我们有解决方案。
- 基于Git，很多东西需要动手，不像Wordpress有强大的后台

大致介绍到此，作为个人博客来说，简洁清爽的表达自己的工作、心得，就已达目标，所以Github Pages是我认为此需求最完美的解决方案了。


## Github Pages + Jekyll 方案适合我吗？

Github Pages + Jekyll 方案的优点：

> Github免费托管源文件，自动编译符合Jekyll规范的网站。
>  引入版本管理，修改网站更加安全方便。
>  支持 [Markdown](https://link.jianshu.com?t=http%3A%2F%2Fsspai.com%2F25137) ，编写具有优美排版的文章。

Github Pages + Jekyll 方案的不足：

> 需要学习一些基础的Git命令。
>  若要自己全权制作主题的话需要懂一点网页开发。
>  由于生成的是静态网页，若要使用动态功能，如评论功能（下文解决），则要使用第三方服务。

所以，如果你只是想做一个分享见闻心得的博客，这个方案非常适合你。


## 相关知识扫盲

#### A（Address）记录

是用来指定主机名（或域名）对应的IP地址记录。用户可以将该域名下的网站服务器指向到自己的web server上。同时也可以设置您域名的二级域名。

#### CNAME

也被称为规范名字。这种记录允许您将多个名字映射到同一台计算机。 通常用于同时提供WWW和MAIL服务的计算机。例如，有一台计算机名为`“host.mydomain.com”`（A记录）。 它同时提供WWW和MAIL服务，为了便于用户访问服务。可以为该计算机设置两个别名（CNAME）：WWW和MAIL。

这两个别名的全称就是`“www.mydomain.com”`和`“mail.mydomain.com”`。实际上他们都指向`“host.mydomain.com”`。 同样的方法可以用于当您拥有多个域名需要指向同一服务器IP，此时您就可以将一个域名做A记录指向服务器IP然后将其他的域名做别名到之前做A记录的域名上，那么当您的服务器IP地址变更时您就可以不必麻烦的一个一个域名更改指向了 只需要更改做A记录的那个域名其他做别名的那些域名的指向也将自动更改到新的IP地址上了。

#### TTL

TTL值全称是“生存时间（Time To Live)”，简单的说它表示DNS记录在DNS服务器上缓存时间。要理解TTL值，请先看下面的一个例子：假设，有这样一个域名`myhost.cnMonkey.com`（其实，这就是一条DNS记录，通常表示在abc.com域中有一台名为myhost的主机）对应IP地 址为1.1.1.1，它的TTL为10分钟。

这个域名或称这条记录存储在一台名为dns.cnMonkey.com的DNS服务器上。现在有一个用户键入一下地址（又称URL）：`http://myhost.cnMonkey.com` 这时会发生什么呢？该 访问者指定的DNS服务器（或是他的ISP,互联网服务商, 动态分配给他的)8.8.8.8就会试图为他解释myhost.cnMonkey.com，当然8.8.8.8这台DNS服务器由于没有包含 myhost.cnMonkey.com这条信息，因此无法立即解析，但是通过全球DNS的递归查询后，最终定位到dns.cnMonkey.com这台DNS服务器， dns.cnMonkey.com这台DNS服务器将myhost.cnMonkey.com对应的IP地址1.1.1.1告诉8.8.8.8这台DNS服务器，然有再由 8.8.8.8告诉用户结果。8.8.8.8为了以后加快对myhost.cnMonkey.com这条记录的解析，就将刚才的1.1.1.1结果保留一段时间，这 就是TTL时间，在这段时间内如果用户又有对myhost.cnMonkey.com这条记录的解析请求，它就直接告诉用户1.1.1.1，当TTL到期则又会重复 上面的过程。

#### 域名分级

子域名是个相对的概念，是相对父域名来说的。域名有很多级，中间用点分开。例如中国国家顶级域名CN，所有以 CN 结尾的域名便都是它的子域。例如：www.zzy.cn 便是 zzy.cn 的子域，而 zzy.cn 是 cn 的子域。

“二级域名”：目前有很多用户认为“二级域名”是自己所注册域名的下一级域名，实际上这里所谓的“二级域名”并非真正的“二级”，而应该称为“次级”(相对次级)

例如你注册的域名是abc.cn来说：CN为顶级域，abc.cn为二级域，www.abc.cn、mail.abc.cn、help.zzy.cn为三级域。

还有一些特殊的二级域被用来作顶级域使用，例如：com.cn、net.cn、org.cn、gov.cn（包括地区域名bj.cn、fj.cn等）。那么此时用户所注册的就应该是三级域了，例如114.com.cn。（备注：www.gov.cn实际上是以gov.cn为后缀的www域名，就是说如果您在域名Whois信息查询中输入gov.cn是查询不到注册信息的因为gov.cn是作为顶级域来使用的域名后缀，真正开放注册的是www.gov.cn）。然而当前有很多用户还是习惯地将可以允许用户注册的域名称为顶级域名，而所注册域名的下一级域名称为“二级域名”，其实从严格意义上来讲这是不对的，所以我们前面会说“子域名”、“二级域名”是相对的概念，准确的应该称为“次级域名”。

#### Jekyll 究竟是什么？

下面是来自 Jekyll中文文档站 的介绍：

> 将纯文本转化为静态网站和博客。
> 简单 -- 无需数据库、评论功能，不需要不断的更新版本——只用关心你的博客内容。
> 静态 -- 只用 Markdown (或Textile)、Liquid、HTML & CSS 就可以构建可部署的静态网站。

Jekyll 是使用Ruby语言开发的一个简单的博客形态静态站点生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何服务器上。截至2017年，Jekyll是最受欢迎的静态网站生成器，主要是由于它被GitHub采用。Jekyll的流行也因为它非常简单，只需要基础的web开发基础，我们可以使用它轻易的把文本转换为自定义的网站/博客。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。

其它类似的静态(博客)网站生成工具还有Hugo（基于go语言）、Hexo（基于Node.js）、Octopress（基于jekyll）、Pelican（基于python）、Middleman（基于Ruby）等很多很多。更多其它请见这里。

# Just do it ！

## 购买、绑定独立域名

虽说Godaddy曾支持过SOPA（禁止网络盗版法案 Stop Online Piracy Act），并且首页放着极其不专业的大胸美女，但是作为域名服务商他做的还不赖，选择它最重要的原因是他支持支付宝，没有信用卡有时真的很难过。流传Godaddy的域名解析服务器被墙掉，导致域名无法访问，后来这个事情在BeiYuu也发生了，不得已需要把域名解析服务迁移到国内比较稳定的服务商处，这个迁移对于域名来说没有什么风险，最终的控制权还是在Godaddy那里，你随时都可以改回去。

域名的购买不用多讲，注册、选域名、支付，有网购经验的都毫无压力，优惠码也遍地皆是。域名的配置需要提醒一下，因为伟大英明的GFW的存在，我们必须多做些事情。

我们最终选择阿里云，阿里云收购了万网，我们可以通过阿里云来购买域名，这里需要先想好自己的域名，不然对于选择强迫症患者将会是异常灾难（.com,.cn,.ink ... ）有众多域名，其中我看好了一个（.me）的后缀的域名，可惜在2017年时依照国家相应网络域名整改办法，许多个性的域名被禁用了，如果是以前购买的话，不影响后面继续续费使用。


## 配置和使用Github

Git是版本管理的未来，他的优点我不再赘述，相关资料很多。推荐这本[Git中文教程](http://git-scm.com/book/zh)。

要使用Git，需要安装它的客户端，推荐在Linux下使用Git，会比较方便。Windows版的下载地址在这里：[Windows版下载地址](https://git-scm.com/)。其他系统的安装也可以参考官方的安装教程。

下载安装客户端之后，各个系统的配置就类似了，我们使用windows作为例子，Linux和Mac与此类似。

在Windows下，打开Git Bash，其他系统下面则打开终端（Terminal）

具体详细操作请参考如何配置Github一章。

由于我本人之前已经配置过Github，SSH的链接已经配置完成，故可输入以下命令查看设置是否成功

```cmd
ssh -T git@github.com
```

反应如下：

```cmd
Hi Kakuoo! You've successfully authenticated, but GitHub does not provide shell access.
```

现在可以通过SSH链接到Github了，但还有一些个人信息需要完善。Git 的用户名字和邮箱需要提交，参照Git 的使用篇。（名字必须自己的名字，真实姓名，而不是Github的昵称）

## 绑定域名到GitHub-Page

如果不绑定域名，那将来域名就会是`kakuoo.github.io`（以我为例），而在自己购买域名`kakuguo.ink`以后就可以将其与Github博客的域名相互绑定，两个域名将指向同一个页面，域名更为美观且有特殊意义。

在windows 的命令行CMD中

```cmd
ping username.github.io
```

可以得到当前github自动为你配置的虚拟主机的地址`185.199.109.153`，然后打开阿里云的网页并登陆，进入`管理控制台`，在`域名解析`的位置点击`新手引导`，在`*记录值`处填入上述地址，点击确定，实现自动配置。

![阿里域名解析设置]({{site.url}}/assets/jekyll_blog_img/AliDNS.png)

接下来在github该博客的仓库里，添加`CNAME`文件（无后缀名），其内容仅为一行，即你购买的域名。

![CNAME]({{site.url}}/assets/jekyll_blog_img/CNAME.png)

之后在仓库的`Settings`的选项中，可以看到绿色的对勾显示为网站已成功设置为相应的域名，实现了绑定。

![github_page]({{site.url}}/assets/jekyll_blog_img/github_page.png)


## 本地环境搭建

本机环境及相关软件版本：

> win10 x64
>
> Ruby 2.5.3(x64)
>
> jekyll 3.8.5

在本文中，我的最终目标是安装Jekyll，Jekyll需要Ruby环境，于是就要先安装Ruby。

我安装的是版本2.5.3，在Windows系统安装Ruby，我是通过RubyInstaller来安装的。

看官网2.4版本与之前2.3的版本的安装是有点差异的：

- 2.3版本的需要下载RubyInstaller和DevKit-mingw64
- 2.4版本之后，需要安装RubyInstaller和MSYS2 toolkit

请看[原文](https://rubyinstaller.org/downloads/)：

> WHICH DEVELOPMENT KIT?
>
> Down this page, several and different versions of Development Kits (DevKit) are listed. Please download the right one for your version of Ruby:
>
> - `Ruby 2.4.0 and newer`: The `MSYS2 DevKit` is downloaded as the `last step of the installation`.（意思是安装RubyInstaller后会提示安装MSYS2，无需像2.3那样另外下载mingw64来安装）
> - Ruby 2.0.0 to 2.3.x (32bits): mingw64-32-4.7.2
> - Ruby 2.0.0 to 2.3.x (64bits): mingw64-64-4.7.2
>
> The RubyInstaller Development Kit (DevKit) is a MSYS/MinGW based toolkit than enables you to build many of the native C/C++ extensions available for Ruby. Starting with Ruby 2.4.0 > it is replaced by the `MSYS2 toolkit`.

### 安装RubyInstaller

下载[RubyInstaller]([http://rubyinstaller.org/downloads/](https://link.jianshu.com/?t=http://rubyinstaller.org/downloads/))并安装，勾选`Add Ruby executable to your PATH`自动添加至系统环境变量，安装至最后会弹出一个CMD窗口，是用来安装MSYS2的，选择`3`并`Enter`回车静静等待安装，整个安装过程比较顺利，但是看MSYS2的官网，还需要升级一下核心的包。 （我本人并未升级核心，不过升级核心的包并不是什么坏事）。

![ruby_install.png]({{site.url}}/assets/jekyll_blog_img/ruby_install.png)

在MSYS2的界面输入升级核心包的命令：`pacman -Syu`（升级核心包），升级过程可能由于网速不稳定不断报错，自行搭梯，多试几次。

如果还有其它问题，官网说可以关闭在重新打开MSYS2，运行命令`pacman -Su`（升级所有包）。

值得一提的是，安装时间很长，至少十几分钟。需要安心等待，这个时间段最好先不要提前进入下列安装步骤，以防出错。

### 安装Jekyll

打开CMD命令行，可以检查一下Ruby和Ruby的包管理器gem（类似于node.js的npm，linux的apt-get，python的anaconda）的安装：

```cmd
PS D:\ProgramData\Git_Repository\Myblogtest> ruby -v
ruby 2.5.3p105 (2018-10-18 revision 65156) [x64-mingw32]
PS D:\ProgramData\Git_Repository\Myblogtest> gem -v
2.7.6
```

由于国内网络原因，导致"http://rubygems.org/"存放在 Amazon S3 上面的资源文件间歇性连接失败。过去可以使用的镜像网站"http://ruby.taobao.org/"也已经无法使用，故可以将它修改为源: "https://gems.ruby-china.com/"。设置一下使用国内的镜像，以防下载过慢而停止：

```cmd
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```

查看gem当前设置的镜像：`gem sources -l`

### 使用gem安装Jekyll：

```cmd
gem install jekyll
```

检查Jekyll的安装：

```cmd
PS D:\ProgramData\Git_Repository\Myblogtest> jekyll -v
jekyll 3.8.5
```

一般在运行某些博客时还需要安装`Bundler`和`jekyll-paginate`，基于报错或提示安装相关缺失的gem。

```cmd
gem install bundler
gem install jekyll-paginate
```

## Jekyll的部署：

参考文档：

>http://jekyllcn.com/
>

### 相关概念解释及答疑：

Jekyll 是基于 Ruby 开发的，所以会用到Ruby的相关命令。

**RubyGems：** 是 Ruby 的一个包管理器，它提供一个分发 Ruby 程序和库的标准格式，还提供一个管理程序包安装的工具。这类似于 Ubuntu 下的apt-get, Centos 的 yum，Python 的 pip。它对应的命令为 `gem`。

**gem命令：**用于构建、上传、下载以及安装Gem包。

**Gem：** 是 Ruby 模块 (叫做 Gems) 的包管理器。其包含包信息，以及用于安装的文件。A Gem is a bundle of code，**Jekyll就是一个Gem，还有大多数的Jekyll插件都是Gem。**

**Gemfile：**包含当前Ruby项目需要运行的Gems列表。当我们需要为Jekyll项目添加插件时，我们可以使用Gemfiles来指定。需要在Gemfiles文件的开头，至少指定一个gem source(指向RubyGems server的url)；可以通过执行命令`bundle init`来创建一个`Gemfile`，其默认source为`https://rubygems.org`详见 [What is a Gemfile](http://tosbourn.com/what-is-the-gemfile/)。

**Bundler：**一个用于管理应用依赖关系的通用工具，它可以跟 RubyGems 搭配使用。通过Bundler程序读取Gemfile然后下载文件中需要的Gems。当我们更改了Gemfile文件后，需要运行`bundle install`命令，该命令生成Gemfile.lock文件，并下载该文件中的gems。

```cmd
bundle install --path vendor/bundle
```

一般我们在初次运行 `bundle install`时会使用`--path`选项，该选项用于指定该项目中使用到的 Gems 的下载安装路径；之后使用`bundle install`时就无需指定`--path`。

Bundler能为我们选择正确的gem版本：Bundler用于在Ruby库中管理Rubygems依赖项。当我们运行`jekyll serve`时，如果我们有一个插件`jekyll-feed`，并且该插件在我们的电脑上有多个版本，此时`jekyll serve`就可能选择了错误的版本；我们可以使用`bundle exec`来解决这个问题，它只会使用Gemfile中该gem需要的版本，示例：使用`bundle exec jekyll serve`代替`jekyll serve`。

执行`bundle exec jekyll serve`或`jekyll serve`后便会生成一个`_site`目录，这就是生成的网站。使用版本控制时建议排除`_site`目录（即在`.gitignore`目录中添加`_site`条款），这样做更加方便往`github`上传代码，不会同步相关中间程序产生的文件改动。Before `building` the site, it actually doesn't exist and is just a bunch of template files。

`jekyll serve`命令的几个作用：

1. 构建您的站点（使用build命令）；
2. 启动开发服务器，并在默认情况下监测文件变化并进行实时构建。任何时候发生变化，它将自动构建您的网站。

`jekyll build`命令：该命令用于创建静态站点(生成`_site`目录)。

`jekyll new`命令：该命令会使用默认的主题，创建一个新的jekyll网站框架(项目)。

### Jekyll插件

当下载使用他人的主题时，运行 `jekyll serve`或`jekyll s`或`jekyll build`命令后，可能出现的错误。

**插件相关的错误：**

第一个错误：某些主题在`_config.yml`配置文件中使用 gems而非plugins。

```cmd
The 'gems' configuration option has been renamed to 'plugins'. Please update your config file accordingly.
只需将： gems 改为 plugins 即可
```

之后的错误，大多数是提示相关插件没有安装，无法找到。没找到就安装该插件。

具体步骤：

- 查看该主题`_config.yml`文件中指定了哪些插件
- 新建Gemfile文件，并在该文件中添加这些插件
- 运行 `bundler install`安装这些插件；或使用`bundle install --path vendor/bundle` 此时推荐将vendor目录添加到 .gitignore 文件中
- 再次运行 `jekyll serve`

**插件的安装方法：**

一般方法，先在Gemfile中添加，然后再在`_config.yml`文件的plugins下添加。

Gemfile文件内容：

```cmd
# frozen_string_literal: true
source "https://gems.ruby-china.com/"

# Kiko Plus 主题所需插件，下面的内容需要根据实际情况进行配置，不能重用
# 并且需要在_config.yml文件的plugins下进行添加
gem 'jekyll-seo-tag'
gem 'jekyll-paginate'
gem 'jekyll-admin'
```

对应的 `_config.yml`文件中的相应内容：

```cmd
plugins:
  - jekyll-feed
  - jekyll-sitemap
  - jemoji
```

**几个插件介绍：**

- Jekyll SEO Tag: 该插件为搜索引擎和社交网络添加元数据标签（meta tags），以更好地索引和显示您网站的内容；使用该插件便可无需手动添加。相关网站 [jekyll-seo-tag](https://github.com/jekyll/jekyll-seo-tag) [Documentation for jekyll-seo-tag (1.2.0)](http://www.rubydoc.info/gems/jekyll-seo-tag/1.2.0)。
- jekyll-paginate: Jekyll提供的分页(paginate)插件，因此您可以自动生成分页列表所需的相应文件和文件夹。开启分页，在_config.yml文件中，通过`paginate:5`来指定每页列出几个文章。[Pagination - Jekyll • Simple, blog-aware, static sites](https://jekyllrb.com/docs/pagination/)
- [Google Analytics（分析） - 网站分析和报告](https://www.google.com/analytics/)：分为两种：一个是用于跟踪Website，一个是用于跟踪Mobile app。创建Google Analytics 帐户需要填写网站名称，网站URL。时区 Reporting Time Zone：China (GMT+08:00) China Time - Beijing 。开启的地址：[Analytics](https://analytics.google.com/analytics/web/provision/?authuser=0#provision/SignUp/)。相关介绍：[Google Analytics （分析） – 帮助中心](https://tumblr.zendesk.com/hc/zh-cn/articles/230864187-Google-Analytics-%E5%88%86%E6%9E%90-) ；[Set up Analytics tracking - Analytics Help](https://support.google.com/analytics/answer/1008080?hl=en)

### `_config.yml`配置文件

1、URL的配置

如果需要部署到GitHub pages，URL一定要配置好。

```
# URL
url:  "https://kakuoo.github.io"
# the base hostname 主机名 & 协议 protocol for your site
# url:  "http://localhost:4000"
# use this url when you develop
baseurl:  "\blog"
# the subpath of your site（网站的子路径,可以为空）, e.g. /blog
# 访问你该网站首页的路径为： url + baseurl = https://fandean.github.io/blog
```

2、Permalinks固定链接

引用变量的方法`:变量`，那么没有`:`前缀的就是固定字符。

在配置文件`_config.yml`中有：

```
permalink:          /:year-:month-:day/:title/
```

以上固定链接表示： 文章生成的html保存在类似`_site/2017-01-19/style-test-ko/index.html`路径中。 其中`_site根目录 = url + baseurl`，`title`目录为 style-stest-ko，它是这篇文章的标题，最终的文章内容将保存在`index.html`文件中。

Jekyll预定义了几个固定链接的格式：比如 `date`表示`/:categories/:year/:month/:day/:title.html` ；我们可以简单的使用`permalink:date`来指定。其中`categories`表示分类，应该需要事先在markdown中指定。

在您的帖子，页面或集合的Front Matter中的permalink会覆盖任何全局设置，例如：

```
---
title: My page title
permalink: /mypageurl/
---
```

该Front Matter中的permalink表示该文章会保存在网站的`/mypageurl/`目录下，这里没有`:`前缀所以是固定字符。

> 参考：[Permalinks - Jekyll • Simple, blog-aware, static sites](https://jekyllrb.com/docs/permalinks/)
>
> 配置文件错误：[Page build failed: Config file error - User Documentation](https://help.github.com/articles/page-build-failed-config-file-error/)
>
> 更多错误： [Troubleshooting GitHub Pages builds - User Documentation](https://help.github.com/articles/troubleshooting-github-pages-builds/)
>
> 有些有 timezone时区设置：示例 上海：`Asia/shanghai` 香港：`Asia/Hong_Kong`

### 进阶：添加评论、分享功能

常见的第三方评论系统有： [Disqus](https://link.jianshu.com/?t=https%3A%2F%2Fdisqus.com%2F)，[多说](https://link.jianshu.com/?t=http%3A%2F%2Fdev.duoshuo.com%2F)。

简单来说是在html文件中嵌入Javascript代码，注册网站后都有较好的指导，并不困难。


## 使用Jekyll：

```cmd
jekyll new myblog
cd myblog
bundle install
bundle exec jekyll serve

# => 打开浏览器 http://localhost:4000（http://127.0.0.1:4000/）
```

值得一提的是，若是自己新建的博客，则不需要`bundle install`命令，如果是从别人的仓库或是从`Jekyll Theme`下载的主题模板,我一般会先删除其自带的`Gemfile.lock`文件，然后均需首先在文件夹中执行`bundle install`命令。尚未阅读相关文档，但我猜测跟`Gemfile`的内容有关，当我执行`bundle exec jekyll`命令后又会生成新的`Gemfile.lock`文件。

一个基本的 Jekyll 网站的目录结构一般是像这样的：

```
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _data
|   └── members.yml
├── _site
└── index.html
```

对应的用途如下：

| 文件 / 目录                                          | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| `_config.yml`                                        | 保存[配置](http://jekyllcn.com/docs/configuration/)数据。很多配置选项都可以直接在命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。 |
| `_drafts`                                            | drafts（草稿）是未发布的文章。这些文件的格式中都没有 `title.MARKUP` 数据。学习如何 [使用草稿](http://jekyllcn.com/docs/drafts/). |
| `_includes`                                          | 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签 `{百分号 include file.ext 百分号}` 来把文件 `_includes/file.ext` 包含进来。 |
| `_layouts`                                           | layouts（布局）是包裹在文章外部的模板。布局可以在 [YAML 头信息](http://jekyllcn.com/docs/frontmatter/)中根据不同文章进行选择。 这将在下一个部分进行介绍。标签 `{ content }` 可以将content插入页面中。 |
| `_posts`                                             | 这里放的就是你的文章了。文件格式很重要，必须要符合:`YEAR-MONTH-DAY-title.MARKUP`。 [永久链接](http://jekyllcn.com/docs/permalinks/) 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。 |
| `_data`                                              | 格式化好的网站数据应放在这里。jekyll 的引擎会自动加载在该目录下所有的 yaml 文件（后缀是 `.yml`, `.yaml`, `.json` 或者 `.csv` ）。这些文件可以经由 ｀site.data｀ 访问。如果有一个 `members.yml` 文件在该目录下，你就可以通过 `site.data.members` 获取该文件的内容。 |
| `_site`                                              | 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 `.gitignore` 文件中。 |
| `.jekyll-metadata`                                   | 该文件帮助 Jekyll 跟踪哪些文件从上次建立站点开始到现在没有被修改，哪些文件需要在下一次站点建立时重新生成。该文件不会被包含在生成的站点中。将它加入到你的 `.gitignore` 文件可能是一个好注意。 |
| `index.html` and other HTML, Markdown, Textile files | 如果这些文件中包含 [YAML 头信息](http://jekyllcn.com/docs/frontmatter/) 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 `.html`, `.markdown`, `.md`, 或者 `.textile` 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。 |
| Other Files/Folders                                  | 其他一些未被提及的目录和文件如 `css` 还有 `images` 文件夹，`favicon.ico` 等文件都将被完全拷贝到生成的 site 中。这里有一些[使用 Jekyll 的站点](http://jekyllcn.com/docs/sites/)，如果你感兴趣就来看看吧。 |

安装完成后，如果你要配置你自己的站点信息，那么你就要修改`_config.yml`这个文件。里面可以配置站点名称，描述，多说，统计，友链等等。


## 应当注意的地方：

如果使用别人的博客模板，记着修改如下东西：

- **统计代码。**如百度统计或者google统计，这样避免麻烦原博主屏蔽一些别的来访id

- **评论代码。**找到别人博客的评论代码，如disqus代码，或者多说评论的代码，否则你的评论会在别人的评论后台提示，自己却看不到提示。

## Error Record

**Msys2无法安装在有空格的目录下。jekyll 的相关操作也无法在有系统文件的文件夹下实现。**Jekyll 3.3.0 之后开始采用 Bundler 方式运行。安装时，需要同时安装 Bundler：

**#Issue：**

```cmd
Please add the following to your Gemfile to avoid polling for changes:
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
```

#Solution：

```cmd
add "gem 'wdm', '~> 0.1.0'"into the Gemfile and the problem was solved.
```

**#Issue：**

```cmd
You have already activated public_suffix 3.0.3, but your Gemfile requires public_suffix 2.0.5. Prepending `bundle exec` to your command may solve this. (Gem::LoadError)
```

#Solution：

```cmd
bundle exec jekyll serve
```

**#Issue：**

```cmd
jekyll 3.1.2 | Error:  Permission denied - bind(2) for 127.0.0.1:4000
```

#Solution：

说明有程序在占用这个本地端口，这时候输入命令

```cmd
$ netstat -ano
```

可以看到如下进程与所占用端口的对应情况，找到本地地址为 `127.0.0.1:4000` 的记录，看到该条记录的PID为6668 (当然你的和我的不一样)。

![PID]({{site.url}}/assets/jekyll_blog_img/PID.png)

输入命令

```cmd
 tasklist /svc /FI "PID eq 6668"
```

该进程的名称就会显示出来:

![PID2]({{site.url}}/assets/jekyll_blog_img/PID2.png)

打开windows的任务管理器，结束它：

![winmanager]({{site.url}}/assets/jekyll_blog_img/winmanager.png)

再次运行 `jekyll serve` 就可以了。

如果一切顺利，通过在浏览器地址栏输入 `http://localhost:4000/` 回车就已经可以看到自己网站的模样啦。
只要 `jekyll serve` 服务开着，你的本地仓库文件有任何更新，本地网站刷新都能马上看到，欧耶！
