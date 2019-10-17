---
title: 各种代理代理情况下网络请求过程分析
date: 2019-10-17
spoiler: 各种代理代理情况下网络请求过程分析
---

最近在折腾透明代理的过程中，对一些基础的网络知识和相关的工具有了更深的了解，下面我们就来分析一下以下几种代理情况网络请求究竟是一个怎样的过程。

1. 未使用任何代理工具
2. 开启代理工具ClashX的系统代理功能并关闭dns功能
3. 开启代理工具ClashX的系统代理功能并打开dns功能
4. 将ClashX与Mellow配合进行透明代理

**注意：**

- 我们主要讨论浏览器发起的网络请求
- 为了方便分析我们不会考虑dns缓存情况
- clash的规则用的是神机规则

## 1. 未使用任何代理工具

浏览器打开baidu.com

```json{1}
1. 首先要进行dns解析获取baidu.com的ip地址
2. 由操作系统调取内置的dns地址并向其发送请求
3. dns请求完成后返回baidu.com的ip地址给浏览器
4. 浏览器向获得的ip地址发起请求进行tcp连接
5. tcp连接成功，开始进行通信
```

## 2. 开启代理工具ClashX的系统代理功能并关闭dns功能

域名规则-直连

```json
DOMAIN-SUFFIX,baidu.com,DIRECT
```

浏览器打开baidu.com

```json{1,3}
1. 由于设置了代理，请求会先经过本地代理服务器，所以不需要dns解析获取ip地址，域名等信息会直接发送到代理服务器
2. 代理服务器获取到请求信息后会根据设置的分流规则进行匹配
3. baidu.com匹配到域名规则-直连，则由操作系统调取内置的dns地址并向其发送请求
4. dns请求完成后返回baidu.com的ip地址给代理服务器
5. 代理服务器向获得的ip地址发起请求进行tcp连接
6. tcp连接成功，开始进行通信
```

上面是baidu.com匹配域名规则走直连的情况，下面我们测试一下走远程服务器代理的情况：

域名规则-代理

```json
DOMAIN-SUFFIX,google.com,PROXY
```

浏览器打开google.com

```json{1,3,4}
1. 由于设置了代理，请求会先经过本地代理服务器，所以不需要dns解析获取ip地址，域名等信息会直接发送到代理服务器
2. 代理服务器获取到请求信息后会根据设置的分流规则进行匹配
3. google.com匹配到域名规则-代理，则直接向远程代理服务器发送tcp请求并携带基本的域名等信息
4. 远程代理服务器获取到域名google.com并进行dns解析，此时的dns解析也被称为远程dns解析
5. 远程dns解析完成，开始进行通信
```

上面是google.com匹配域名规则走代理的情况，有时候我们还会遇到ip规则的情况：

ip规则-代理

```json
IP-CIDR,35.190.247.0/24,PROXY
```

浏览器打开google.com

```json{1,3,4}
1. 由于设置了代理，请求会先经过本地代理服务器，所以不需要dns解析获取ip地址，域名等信息会直接发送到代理服务器
2. 代理服务器获取到请求信息后会根据设置的分流规则进行匹配
3. google.com没有匹配到任何域名规则，由于存在ip规则的原因，所以需要先进行本地dns解析获取到ip，然后将获取的ip与ip规则进行匹配，经过匹配走代理，则直接向远程代理服务器发送tcp请求并携带基本的域名等信息
4. 远程代理服务器获取到域名google.com并再次进行dns解析
5. 远程dns解析完成，开始进行通信
```

## 3. 开启代理工具ClashX的系统代理功能并打开dns功能

clash的dns配置

```json
dns:
  enable: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip # redir-host 或 fake-ip
  nameserver:
    - 119.29.29.29
    - 223.5.5.5
    - 114.114.114.114
```

**注意：**

这里的listen和enhanced-mode在当前情况下不需要关注

域名规则-直连

```json
DOMAIN-SUFFIX,baidu.com,DIRECT
```

浏览器打开baidu.com

```json{1,3}
1. 由于设置了代理，请求会先经过本地代理服务器，所以不需要dns解析获取ip地址，域名等信息会直接发送到代理服务器
2. 代理服务器获取到请求信息后会根据设置的分流规则进行匹配
3. baidu.com匹配到域名规则-直连，由于clash的dns功能开启，则clash会向nameserver中的所有dns地址同时发送请求并获取最先返回的ip地址
4. 代理服务器向获得的ip地址发起请求进行tcp连接
5. tcp连接成功，开始进行通信
```

浏览器打开google.com

```json{1,3,4}
1. 由于设置了代理，请求会先经过本地代理服务器，所以不需要dns解析获取ip地址，域名等信息会直接发送到代理服务器
2. 代理服务器获取到请求信息后会根据设置的分流规则进行匹配
3. google.com匹配到域名规则-代理，则直接向远程代理服务器发送tcp请求并携带基本的域名等信息
4. 远程代理服务器获取到域名google.com并进行dns解析，此时的dns解析也被称为远程dns解析
5. 远程dns解析完成，开始进行通信
```

上面是google.com匹配域名规则的情况，有时候我们还会遇到ip规则的情况：

ip规则-代理

```json
IP-CIDR,35.190.247.0/24,PROXY
```

浏览器打开google.com

```json{1,3,4}
1. 由于设置了代理，请求会先经过本地代理服务器，所以不需要dns解析获取ip地址，域名等信息会直接发送到代理服务器
2. 代理服务器获取到请求信息后会根据设置的分流规则进行匹配
3. google.com没有匹配到任何域名规则，由于存在ip规则的原因，所以需要先进行本地dns解析获取到ip，然后将获取的ip与ip规则进行匹配，由于clash的dns功能开启，则clash会向nameserver中的所有dns地址同时发送请求并获取最先返回的ip地址，经过匹配走代理，则直接向远程代理服务器发送tcp请求并携带基本的域名等信息
4. 远程代理服务器获取到域名google.com并再次进行dns解析
5. 远程dns解析完成，开始进行通信
```

## 4. 将ClashX与Mellow配合进行透明代理

由于clash的tun功能还未实现，所以进行透明代理的话这里配合mellow使用，主要流程就是：

```json
浏览器发起请求->mellow->clash->direct
```

因为所有的请求都会经过mellow，所以如果不进行特殊配置，clash发出的请求还会经过mellow再次到clash，为了避免这种回环情况，mellow配置如下：

```json{6-17,25-32}
{
  "log": {
    "loglevel": "debug"
  },
  "outbounds": [
    {
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "127.0.0.1",
            "port": 1080
          }
        ]
      },
      "tag": "clash"
    },
    {
      "protocol": "freedom",
      "tag": "direct"
    }
  ],
  "routing": {
    "rules": [
      {
        "app": [
          "ClashX",
          "clash"
        ],
        "type": "field",
        "outboundTag": "direct"
      }
    ]
  }
}
```

经过上面配置，经过mellow的请求会直接发送到clash的1080 socks端口，clash发起的请求会直接发送出去，回环问题解决。

由于开启clashx的系统代理后，浏览器的请求会跟之前一样，会先发送到clash，所以这里我们将clash的系统代理关闭，开启mellow，来分析下网络请求过程：

域名规则-直连

```json
DOMAIN-SUFFIX,baidu.com,DIRECT
```

浏览器打开baidu.com

```json{1,2,4,7,10}
1. 由于未开启系统代理，浏览器正常发起请求，首先进行dns解析
2. 由于透明代理的原因，dns的请求流量会被路由到TUN接口，被mellow拦截，由于mellow的配置文件中未使用自定义的dns，所以利用本地dns进行解析，解析之后将ip返回给浏览器
3. 浏览器向ip发起请求进行tcp连接
4. tcp请求被mellow拦截并转发到clash，因为之前进行dns拦截时mellow会利用sniffing嗅探出dns请求的域名信息，所以这里转发给clash的请求会将域名信息带上
5. clash获取到请求信息后会根据设置的分流规则进行匹配
6. baidu.com匹配到域名规则-直连，则由操作系统调取内置的dns地址并向其发送请求
7. clash发起的dns请求同样被mellow拦截，但由于mellow的规则中对clash设置了直连，所以直接发送出去
8. dns请求完成后返回baidu.com的ip地址给clash
9. clash向获得的ip地址发起请求进行tcp连接
10. 同样，clash发起的tcp请求同样被mellow拦截，但由于mellow的规则中对clash设置了直连，所以直接发送出去
11. tcp连接成功，开始进行通信
```

**注意：**

当系统dns为路由器网关或私有地址时，此时dns请求流量就不会被路由到TUN接口，mellow是拦截不到的，也就是说最终到达clash的请求是没有域名信息的，只能进行ip规则匹配

## 结论

经过上面的分析可以得出以下几点：

- 未开启代理的情况下，网络请求需要先进行dns解析获取到ip，然后根据ip进行tcp连接
- 开启代理后，网络请求直接发送到代理服务器，不需要进行dns解析
- clah是进行本地dns解析还是远程dns解析主要由以下情况：
  - 本地：匹配域名规则走直连、进行ip规则匹配
  - 远程：匹配域名规则走远程代理、进行ip规则匹配后还需要走远程代理
- clash的dns功能开启后，涉及到dns解析的操作不会使用系统dns而是会使用nameserver中的dns
