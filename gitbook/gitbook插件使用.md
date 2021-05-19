# 配置gitbook插件

gitbook默认只有一些插件，其他插件需要自己安装，安装插件很简单，在content目录下配置一个book.json即可

```
{
    "title": "运维操作指南",
    "description": "运维操作指南",
    "author": "vishon",
    "gitbook": ">= 3.2.2",
    "language": "zh-hans",
    "links": {
        "sidebar": {
            "聂伟星个人博客": "https://www.niewx.cn"
        }
    },
    "plugins": [
        "github",
        "editlink",
        "-lunr",
        "-search",
        "search-plus",
        "tbfed-pagefooter",
        "splitter",
        "page-toc-button",
        "back-to-top-button",
        "-lunr", "-search", "search-plus",
        "github-buttons@2.1.0",
        "favicon@^0.0.2",
        "3-ba",
        "disqus",
        "theme-default"
    ],
    "pdf": {
        "toc": true,
        "pageNumbers": true,
        "fontSize": 11
    },
    "pluginsConfig": {
        "github": {
            "url": "https://github.com/nieweixing"
        },
        "editlink": {
            "base": "https://github.com/nieweixing/mybook/tree/gh-pages",
            "label": "编辑本页"
        },
        "tbfed-pagefooter": {
          "copyright":"&copy vishon",
          "modify_label": "Updated at",
          "modify_format": "YYYY-MM-DD HH:mm:ss"
        },
        "image-captions": {
            "caption": "图片 - _CAPTION_"
        },
        "github-buttons": {
            "repo": "nieweixing/mybook",
            "types": ["star"],
            "size": "small"
        },
        "favicon": {
            "shortcut": "favicon.ico",
            "bookmark": "favicon.ico"
        },
        "disqus": {
            "shortName": "nieweixing-github-io"
        },
        "3-ba": {
            "token": "014238987a800856443fcb5e465f4cdd"
        }
    },
    "generator": "site"
}
```

配置好之后，需要执行npm install安装下插件，安装完成后会将插件放在node_modules目录，这里如果下载插件很慢，可以直接到我的github目录下拷贝对应的包<https://github.com/nieweixing/mybook/tree/gh-pages/gitbook>

```
gitbook install ./content 
```

# 配置discuss评论系统

这里可以给gitbook配置disscus评论系统

* 注册 disqus账号 https://disqus.com/ 
* 右上角 Setting --> Add Disqus To Site --> 最下面 GET STARTED --> I want to install ... 
* 填写网站 name 和分类 --> Create Site
* 填写 Website URL如  https://www.niewx.cn/mybook

配置好之后再book.json的disqus字段配置你创建的shortname，最后在gitbook目录下执行发布命令发布到github，这样我们就可以使用discuss评论功能了

![upload-image](image/Snipaste_2021-05-19_20-34-28.png) 