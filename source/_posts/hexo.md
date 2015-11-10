title: 搭建一个独立的博客
date: 2015-10-07 23:25:44
tags: ["hexo","next"]
categories: ["Node.js"]
---
hello，我是沙沙声，一直没有养成写博客的习惯 。当前正好有时间，决定现在开始养成写博客的习惯
第一篇就从搭建一个静态博客开始吧，我使用的是Hexo生成静态页面，部署到 Github Pages上

<h2 id="theory">原理</h2>
>详见[官方地址](https://help.github.com/categories/github-pages-basics/)

Github会将以用户名命名的仓库中的master分支的文件生成Github Pages 站点

>如用户名为username，在username仓库下的master分支下的文件会被Githubt生成Github Pages 站点，地址为<https://username.github.io>

印象中Github生成是按Jekyll格式生成的（除了当前用户名命名的仓库，其它仓库需要分支为gh-pages)

>如仓库名project生成的Github Pages地址为<https://username.github.io/project>）

Jekyll提供舒适的文档，可以在[Jekyll提供的文档](http://jekyll.bootcss.com/)中获取更多详情
Ruby配置稍微有点麻烦,所以我并没有使用Jekyll，而是使用Hexo

现在开始配置自己的静态博客了，现在开始一步步搭建吧
<h2 id="depend">解决依赖</h2>
hexo使用的是node.js，为了部署到Github,我们还需要安装Git

+ 注册Github:
在https://github.com/ 注册自己的帐号
+ 安装Git与配置：
Fedora可以直接通过dnf（yum）安装
`dnf install git`或者`yum install git`
或到[官网](http://www.git-scm.com/download) 下载对应的安装文件

>接下来的配置都是以命令的形式生成，如果在windows下，建议使用完成安装后的Git Bash来完成所有命令

 + 配置用户名 `git config --global user.name "dengzf"`
 + 配置邮箱 `git config --global user.email "dengzhengfeng1991@126.com"`
 + 生成密钥 `ssh-keygen -t rsa -C "gudujianjsk@gmail.com"`

>生成过程中会提示密钥的生成地址与密码，密码可为空不填写)，生成的公钥在对应home下的.ssh目录下（windows对应的用户目录下）的id_rsa.pub文件，用本文格式打开以备下一阶段使用

+ 配置Github的ssh key
登录Github，点击右上角图标->Settings->SSH keys->Add SSH key 在Key一栏输入id_rsa.pub文件里的内容，然后Add key

![配置SSH key](/img/hexo/sshkey.png)
测试是否配置成功 ssh -T git@github.com

+ 安装node.js与npm:
Fedora可以直接通过dnf（yum）安装 
`dnf install node npm`或者`yum install node npm`
或到[官网](https://nodejs.org/en/download) 下载对应的安装文件

+ 安装Hexo:
`npm install -g hexo-cli`

准备工作已就绪，接下来就可以生成静态博客了
<h2 id="init">生成博客</h2>
1. 生成博客项目：hexo init blog (如果第一次生成，进入目录后需要`npm install`)
2. 生成静态文件：`hexo g`
3. 本地运行项目：`hexo s`

这时后就可以在本地 <http://127.0.0.1:4000/>查看Hello World了
>几个Hexo常用命令与简写
`hexo new "文章"` 简写 `hexo n "文章"` #新建文章
`hexo generate` 简写 `hexo g` #生成静态文件
`hexo server` 简写 `hexo s` #启动服务预览
`hexo deploy` 简写 `hexo d` #部署

<h2 id="deployer">推送到Github</h2>
>生成的静态博客最终是要推送到Github，首先创建名为用户名的仓库如我的用户名是dengzhengfeng就创建名为dengzhengfeng的仓库

1. 安装git插件
`npm install hexo-deployer-git --save`
2. 配置文件 _config.yml
找到deploy属性，修改成对应的自己的仓库
`deploy:`
&nbsp;&nbsp;`type: git`
&nbsp;&nbsp;`repositor: git@github.com:dengzhengfeng/dengzhengfeng.github.io.git`
&nbsp;&nbsp;`branch: master`
3. 最后部署`hexo d`

>注意：所有的分号后都有一个空格，完成部署后Github Pages生成页面需要一定的时间，如用户名为username，最终生成页面的地址为<https://username.github.io>

<h2 id="article">生成一篇文章</h2>
1. 生成markdown文件
`hexo n "测试文章"`

>会在source/_posts/目录下生成"测试文章.md"，可以在文件中编写符合[Markdown](http://wowubuntu.com/markdown/) 语法的代码，完成后使用hexo g 生成博客文件hexo s 预览，完成后可以hexo d 推送到Github完成

<h2 id="next">安装Next主题</h2>
>主题能让我们更方便的管理博客（以[Next](https://github.com/iissnan/hexo-theme-next)为例）

1. 在博客对应目录下
 `git clone https://github.com/iissnan/hexo-theme-next themes/next`
2. 修改站点配置文件 _config.yml（注意 所有:后都有一个空格）
`theme: next`
3. 将 language 设置成你所需要的语言
`language: zh-Hans`

>完成后就可以看到最新的主题效果，更多配置可以在[Next官网](https://github.com/iissnan/hexo-theme-next)查询到，或使用其它主题，完成独立博客的搭建

<h2 id="duoshuo">添加评论功能</h2>
1.注册[多说](http://duoshuo.com/)
2.创建站点，填写表单，多说域名 这一栏填写的为 duoshuo_shortname
3.创建站点完成后在 站点配置文件 _config.yml 中新增 duoshuo_shortname 字段，值设置成上一步中的值。