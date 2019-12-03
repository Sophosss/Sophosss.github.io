---
title: GitHub pages + Jekyll博客
author: Sophosss
layout: post
---
### 快速搭建：

不用安装各种文件，只需拥有github账户，本地提供git bash工具。

- 前往[Jekyll]([http://jekyllthemes.org](http://jekyllthemes.org/) )挑选你喜爱的主题模板。![theme](https://Sophosss.github.io/assets/images/1.png)

- 选择homepage进入该主题的GitHub主页，点击fork，自动在你的GitHub里创建一个仓库。

- 修改仓库名称为yourname.github.io(用户名建议小写)。

  ![theme](https://Sophosss.github.io/assets/images/2.png)

- 修改_config.yml文件，将`theme:name`修改为`remote_theme:author/name`，让GitHub Pages支持GitHub上的各种开源主题。

- 最后设置下`base_url`和`url`就大功告成啦，可以将`base_url`为空，`url`设置为你自己的主页。

  ![theme](https://Sophosss.github.io/assets/images/3.png)

### 快速使用：

接下来就可以开始个人博客之旅了，使用markdown编写一篇博客放到_posts文件夹内(注意文件命名格式哦)，使用git bash上传更新到你的GitHub。

刷新一下你的主页看看吧~