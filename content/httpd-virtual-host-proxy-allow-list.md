---
title: "httpd VirtualHost Proxy正向代理白名单"
date: 2019-12-11T10:52:57+08:00
draft: true
---
# httpd VirtualHost Proxy正向代理白名单
因故需要使用httpd搭建一个正向代理服务器，并且对代理的域名进行限制，只允许代理其中一些域名。  
配置过程中发现当在VirtalHost下设置ProxyRequests On之后，Proxy和ProxyMatch指令的行为都不符合预期。  
具体表现为会代理任意域名，而不是只代理匹配通配符或者正则表达式的域名。  
看了[apache expressions的文档](https://httpd.apache.org/docs/2.4/expr.html)发现可以通过expressions限制HTTP_HOST来达到对代理的域名进行限制。  
示例如下，特别注意对于https的域名，host需要加上端口号443。

```
<VirtualHost *:8888>
        ProxyRequests On
        <Proxy "*">
                Require expr %{HTTP_HOST} in {'allowed.host1','allowed.https.host2:443'}
        </Proxy>
</VirtualHost>
```
