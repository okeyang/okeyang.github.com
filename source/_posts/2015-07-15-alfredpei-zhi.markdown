---
layout: post
title: "Alfred配置"
date: 2015-07-15 22:45
comments: true
categories: Tools
---
一直使用[Alfred](http://www.alfredapp.com)这款神器，为了将效率更提升一步，最近一狠心，买了的[Powerpack](http://www.alfredapp.com/powerpack/)，对于每项配置的具体介绍推荐读一下这篇文章：[丢掉鼠标－Mac神软Alfred使用手册1](http://wellsnake.com/jekyll/update/2014/06/15/001/)。下面重点说一下我的`Web Search`和`Workflows`配置。

## Web Search
<img src="/images/2015/alfred_web_search.png">

尽可能把会使用到的搜索地址添加进来，这样就可以尽可能使用`Alfred`作为入口，提升效率，以下是我自行添加的搜索地址URL

##### Github
`https://github.com/search?utf8=%E2%9C%93&q={query}` 

需勾选`Encode spaces as +`

##### Stackoverflow
`http://stackoverflow.com/search?q={query}`

需勾选`Encode spaces as +`

##### 百度
`https://www.baidu.com/s?wd={query}`

##### 百度百科
`http://baike.baidu.com/search/word?word={query}`

##### 百度图片
`http://image.baidu.com/i?z=&s=1&ct=201326592&cl=2&lm=-1&tn=baiduimage&ie=utf-8&word={query}`

##### 知乎
`http://www.zhihu.com/search?q={query}&type1=all`

##### 天猫
`http://list.tmall.com/search_product.htm?q={query}`

##### 淘宝
`http://s.taobao.com/search?q={query}`

##### 京东
`http://search.jd.com/Search?keyword={query}&enc=utf-8`

## Workflows
#### [Google Suggest](https://github.com/yang6512/alfredworkflows/raw/master/Google%20Suggest.alfredworkflow)
直接使用的Examples里面的，在此基础上稍微做了下改进，修复了一些bug，还支持`tab`自动补全，可以在搜索建议结果未返回时直接搜索内容。

<img src="/images/2015/alfred_google_suggest.png">

如果使用代理服务器，可以通过修改源码实现，点击如下所示`Open workflow folder`

<img src="/images/2015/alfred_open_workflow_folder.png">

然后编辑workflows.php，找到
``` php
public function request( $url=null, $options=null )
{
    ...
    $defaults = array(                          // Create a list of default curl options
        CURLOPT_RETURNTRANSFER => true,         // Returns the result as a string
        CURLOPT_URL => $url,                    // Sets the url to request
        CURLOPT_FRESH_CONNECT => true,
        // 以下为自行添加参数
        CURLOPT_PROXY => "127.0.0.1",           // 代理服务器地址
        CURLOPT_PROXYPORT => "10010",           // 代理接口
        CURLOPT_PROXYTYPE => CURLPROXY_SOCKS5   // 代理类型，默认为HTTP代理，这里是使用socks5代理
        // 如果还有用户名密码，则添加：CURLOPT_PROXYUSERPWD => "user:password"
    );
    ...
}
```
#### [Baidu Suggest](https://github.com/yang6512/alfredworkflows/raw/master/Baidu%20Suggest.alfredworkflow)
这个是照虎画猫自己写的，使用和`Google Suggest`类似。

<img src="/images/2015/alfred_baidu_suggest.png">

**以上两个`Workflow`是我最频繁使用的功能，相当于直接把Google和Baidu网站上的搜索框放进了`Alfred`，利用搜索建议大大提升搜索效率，强烈推荐。**

#### [IP Address](http://dferg.us/ip-address-workflow/)
获取本地IP和公网IP。

<img src="/images/2015/alfred_ip_address.png">

#### [VPN Connections](http://www.packal.org/workflow/vpn-connections)
快速打开/关闭VPN。

<img src="/images/2015/alfred_vpn_connections.png">

#### [Colors](http://www.packal.org/workflow/colors)
很方便的转换Color值，如从Hex转到UIColor的rgb格式，可以识别NSColor和UIColor的方法，非常方便。

<img src="/images/2015/alfred_colors.png">

#### [Encode / Decode (v1.8)](https://github.com/willfarrell/alfred-encode-decode-workflow)
方便的进行URL、HTML、Base64的encode和decode。

<img src="/images/2015/alfred_encode_decode.png">

## Workflows网站
http://www.packal.org/

https://github.com/zenorocha/alfred-workflows