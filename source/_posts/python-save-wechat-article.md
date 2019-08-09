---
title: 如何使用 Python 爬取微信公众号文章
date: 2019-08-09 10:26:04
tags: Python
---

我比较喜欢看公众号，有时遇到一个感兴趣的公众号时，都会感觉相逢恨晚，想一口气看完所有历史文章。但是微信的阅读体验挺不好的，看历史文章得一页页的往后翻，下一次再看时还得重复操作，很是麻烦。

于是便想着能不能把某个公众号所有的文章都保存下来，这样就很方便自己阅读历史文章了。

话不多说，下面我就介绍如何使用 Python 爬取微信公众号所有文章的。

主要有以下步骤：

1 使用 Fiddler 抓取公众号接口数据

2 使用 Python 脚本获取公众号所有历史文章数据

3 保存历史文章

## Fiddler 抓包

Fiddler 是一款抓包工具，可以监听网络通讯数据，开发测试过程中非常有用，这里不多做介绍。没有使用过的可以查看这篇文章，很容易上手。

https://blog.csdn.net/jingjingshizhu/article/details/80566191

接下来，使用微信桌面客户端，打开某个公众号的历史文章，这里以我的公众号举例，如下图。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/wechat_article.png)

如果你的 fiddler 配置好了的话，能够看到如下图的数据。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/fiddler_wechat.png)

图中包含抓取的 url、一些重要的参数和我们想要的数据。

这些参数中，`offset` 控制着翻页，其他参数在每一页中都是固定不变的。

接口返回的数据结构如下图，其中 `can_msg_continue` 字段控制着能否翻页，1 表示还有下一页，0 表示没有已经是最后一页了。 `next_offset` 字段就是下一次请求的 `offset` 参数。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/wechat_json.png)

##  构造请求，获取数据

接下来我们的目标就是根据 url 和一些参数，构建请求，获取标题、文章 url 和日期等数据，保存数据。

保存数据一种是使用 pdfkit 将 文章 url 保存为 pdf 文件；另一种是先保存 html 文件，然后将 html 制作成 chm 文件。

1 将 文章 url 保存为 pdf 文件，关键代码如下：

```python
def parse(index, biz, uin, key):
    # url前缀
    url = "https://mp.weixin.qq.com/mp/profile_ext"

    # 请求头
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 "
                      "Safari/537.36 MicroMessenger/6.5.2.501 NetType/WIFI WindowsWechat QBCore/3.43.901.400 "
                      "QQBrowser/9.0.2524.400",
    }

    proxies = {
        'https': None,
        'http': None,
    }

    # 重要参数
    param = {
        'action': 'getmsg',
        '__biz': biz,
        'f': 'json',
        'offset': index * 10,
        'count': '10',
        'is_ok': '1',
        'scene': '124',
        'uin': uin,
        'key': key,
        'wxtoken': '',
        'x5': '0',
    }

    # 发送请求，获取响应
    response = requests.get(url, headers=headers, params=param, proxies=proxies)
    response_dict = response.json()

    print(response_dict)

    next_offset = response_dict['next_offset']
    can_msg_continue = response_dict['can_msg_continue']

    general_msg_list = response_dict['general_msg_list']
    data_list = json.loads(general_msg_list)['list']

    # print(data_list)

    for data in data_list:
        try:
            # 文章发布时间
            datetime = data['comm_msg_info']['datetime']
            date = time.strftime('%Y-%m-%d', time.localtime(datetime))

            msg_info = data['app_msg_ext_info']

            # 文章标题
            title = msg_info['title']

            # 文章链接
            url = msg_info['content_url']

            # 自己定义存储路径（绝对路径）
            pdfkit.from_url(url, 'C:/Users/admin/Desktop/wechat_to_pdf/' + date + title + '.pdf')

            print(title + date + '成功')

        except:
            print("不是图文消息")

    if can_msg_continue == 1:
        return True
    else:
        print('爬取完毕')
        return False
```

2 保存 html 文件，关键代码如下

```python
def parse(index, biz, uin, key):

    # url前缀
    url = "https://mp.weixin.qq.com/mp/profile_ext"

    # 请求头
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 "
                      "Safari/537.36 MicroMessenger/6.5.2.501 NetType/WIFI WindowsWechat QBCore/3.43.901.400 "
                      "QQBrowser/9.0.2524.400",
    }

    proxies = {
        'https': None,
        'http': None,
    }

    # 重要参数
    param = {
        'action': 'getmsg',
        '__biz': biz,
        'f': 'json',
        'offset': index * 10,
        'count': '10',
        'is_ok': '1',
        'scene': '124',
        'uin': uin,
        'key': key,
        'wxtoken': '',
        'x5': '0',
    }

    # 发送请求，获取响应
    reponse = requests.get(url, headers=headers, params=param, proxies=proxies)
    reponse_dict = reponse.json()

    # print(reponse_dict)
    next_offset = reponse_dict['next_offset']
    can_msg_continue = reponse_dict['can_msg_continue']

    general_msg_list = reponse_dict['general_msg_list']
    data_list = json.loads(general_msg_list)['list']

    print(data_list)

    for data in data_list:
        try:
            datetime = data['comm_msg_info']['datetime']
            date = time.strftime('%Y-%m-%d', time.localtime(datetime))

            msg_info = data['app_msg_ext_info']

            # 标题
            title = msg_info['title']

            # 内容的url
            url = msg_info['content_url'].replace("\\", "").replace("http", "https")
            url = html.unescape(url)
            print(url)

            res = requests.get(url, headers=headers, proxies=proxies)
            with open('C:/Users/admin/Desktop/test/' + title + '.html', 'wb+') as f:
                f.write(res.content)

            print(title + date + '成功')

        except:
            print("不是图文消息")

    if can_msg_continue == 1:
        return True
    else:
        print('全部获取完毕')
        return False
```

## 保存文章

**保存为 pdf 文件**，用到了 python 的第三方库 pdfkit 和 wkhtmltopdf。

安装 pdfkit：

```python
pip install pdfkit
```

安装 wkhtmltopdf：

下载地址：

https://wkhtmltopdf.org/downloads.html

安装后将 wkhtmltopdf 目录下的 bin 添加到环境变量中。

**保存为 chm 文件**，可以下载 Easy CHM ，使用这个软件可以将 html 制作成 chm，使用教程网上比较多。

下载地址：

http://www.etextwizard.com/cn/easychm.html

效果图：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/chm.png)

**pdf 和 chm 对比**：

pdf 支持多终端，阅读体验好，但是有个大坑，就是微信文章保存的 pdf 没有图片，很影响阅读体验，暂未找到解决办法。

chm 的好处是可以建立索引，查看文章方便。一个公众号制作成一个 chm 文件，管理方便。不会出现图片不显示问题。

所以推荐将爬取到的公众号文章保存为 chm 文件，方便阅读。

需要完整代码和文中相关软件的朋友，可以关注公众号「JaqenAndroid」，后台回复关键词 ”**公众号文章**“ 获取。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/qrcode_JaqenAndroid.jpg)



