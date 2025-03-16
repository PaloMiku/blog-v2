---
title: 编写1Panel应用商店第三方程序
description: 1Panel 是我目前在使用的一个服务器管理面板，它的社区版开源且免费，整体面板设计简洁快速，不需要登录，在基本体验上似乎是比隔壁宝塔要强一些。
date: 2024-10-17 19:24:26
categories: [技术探索]
tags: [1Panel]
---

## 前言

1Panel 是我目前在使用的一个服务器管理面板，它的社区版开源且免费，整体面板设计简洁快速，不需要登录，在基本体验上似乎是比隔壁宝塔要强一些。

不过他被很多小白和部分用户不喜爱的原因是它的应用是通过 Docker 容器化进行部署的。

采用 Docker 部署的坏处是应用整体内存占用会略高一些，而且配置容器组网是劝退很多小白的原因，配置不好，应用连不上数据库，网站反代不对等等。

但是 Docker 容器化部署的好处也是不少的，比如没有构建应用这一步，就使得应用在拉取完毕镜像以后即刻就能启动，就跟二进制应用文件差不多的感觉，不需要等上很久，也不需要太多服务器性能用于构建，在部署 Node.js 应用时效果尤为显著。而且配置文件简单，只需要了解 Docker Compose 配置文件格式就能上手，整体体验对于大规模部署是很好的。

当然 1Panel 也是知道小白们可能玩不明白 Docker 的，所以它还有应用商店，里面都是预先写好的 Docker Compose 文件模板，点击安装就能够一键部署，并且都自动为你做好网络设置，同时你还可以设计自己的应用上传到服务器，也可以一键安装，这样就出现了 1Panel 官方应用商店和第三方应用商店的存储库，只需要拉取仓库到服务器就能畅享海量应用。

实际上 1Panel 应用的结构很简单，只需要几个文件，就能够设计好你想部署的应用，今天我也来简单讲一下如何设计这个应用。

## 注意

值得注意的事情是，1Panel 应用商店的本质其实就是一个个 Docker Compose 编排文件的图形体现，所以实际上需要你的应用有公共可访问的 Docker 镜像才能正常使用。

## 格式

### 初始化模板

从`1panel 1.3`及以上版本开始，可以在安装了 1Panel 的服务器使用`1panel app init <应用的key> <应用的版本>`指令来快速初始化应用模板文件（注意不是 `1pctl`命令）

### 文件夹格式

应用的文件夹格式大概是长下面这个样子：

```
├──halo // 承载应用的文件夹，这里以halo为例
    ├── logo.png // 应用的 logo
    ├── data.yml // 应用声明文件
    ├── README.md // 应用的 README 文件
    ├── 2.2.0 // 应用版本 注意不要以 v 开头
    │&nbsp;&nbsp; ├── data.yml // 应用的参数配置，下面有详细介绍
    │&nbsp;&nbsp; ├── data // 挂载出来的目录
    |   ├── scripts // 脚本目录 存放 init.sh upgrade.sh uninstall.sh
    │&nbsp;&nbsp; └── docker-compose.yml // docker-compose 文件
    └── 2.3.2
        ├── data.yml
        ├── data
        └── docker-compose.yml
```

### 应用 Logo

用于在应用概览和应用详情页面展示，应用的 Logo 为 png 格式，长宽比最好是`180 * 180 px`，把它放在应用的根目录下，例如`/halo/logo.png`，不要使用诸如 webp，jpg，svg 这样的格式。

### 应用声明文件

声明你的应用的基本信息，让 1Panel 知道你的应用的一些基本信息，比如叫什么名字，干什么用的等等，是位于应用根目录下的`data.yml`文件，格式差不多如下：

```
# 下边这一部分推荐填写，虽然 1Panel 的应用商店官方 Wiki 没有声明它到底有何用
name: Halo
tags:
  - 建站
title: 强大易用的开源建站工具
type: 建站
description: 强大易用的开源建站工具
# 下边这一部分为应用声明信息部分，注意填写正确！
additionalProperties:  #固定参数
    key: halo   #应用的 key，仅限英文，用于在 Linux 创建文件夹
    name: Halo  #应用名称
    tags:
        - WebSite #应用标签，可以有多个，请参照标签列表
    shortDescZh: 强大易用的开源建站工具 #应用中文描述，不要超过 30 个字
    shortDescEn: Powerful and easy-to-use open source website builder #应用英文描述
    type: website  #应用类型，区别于应用分类，只能有一个，请参照下方的类型列表
    crossVersionUpdate: true  #是否可以跨大版本升级
    limit: 0  #应用安装数量限制，0 代表无限制
    website: https://halo.run/  #应用的官网地址
    github: https://github.com/halo-dev/halo #应用 GitHub 的仓库地址
    document: https://docs.halo.run/ #应用的文档地址
```

我们对必填应用声明信息部分从上往下进行详细的介绍：

### key

诸如 halo 这样的名字，它其实就是你应用文件夹的名字，注意此处一定要与应用文件夹的名字保持一致，不然会导致应用无法正常部署安装。

### name

应用的名称部分，用于 1Panel 商店列表中展示和应用详情上方展示，填写好了能提高应用的辨认度。

### tags

应用的标签，决定了应用在应用商店所属的分类，可以填写多个。

标签列表和其对应的分类如下：

| 值 | 所属分类 | 值 | 所属分类 | 值 | 所属分类 |
| --- | --- | --- | --- | --- | --- |
| WebSite | 建站 | Storage | 云存储 | Email | 邮件服务 |
| Server | Web 服务器 | AI | AI/大模型 | Game | 休闲游戏 |
| Runtime | 运行环境 | BI | BI |  |  |
| Database | 数据库 | Security | 安全 |  |  |
| Tool | 实用工具 | DevTool | 开发工具 |  |  |
| DevOps | DevOps | Middleware | 中间件 |  |  |
| Local | 本地 | Media | 多媒体 |  |  |

### shortDescZh/En

应用的中文/英文描述，用于应用在应用商店列表中和应用详情页上方快速介绍应用，不要太长（中文不超过 30 字）

### type

应用的用途分类，这里主要决定了 1Panel 会怎么对待你的应用，比如`website`类型的可以在其网站功能中一键进行反代和分配域名。

| 值 | 说明 |
| --- | --- |
| website | website 即网站应用程序类型，支持在网站中一键部署和反代，WordPress 和 Halo 都是此 type |
| runtime | mysql openresty redis 等类型的应用，倾向于应用的关键运行环境 |
| tool | phpMyAdmin redis-commander jenkins 等类型的应用，倾向于应用的维护工具等 |

### crossVersionUpdate

决定了应用是否可以跨大版本进行升级，一般来说都可以吧。

### limit

决定了这个应用最多能部署多少个，一般为`0`无限制，但是有些应用它确实不能重复安装，比如 Openresty 这样的反向代理服务。

### website

应用的官网地址，但不是所有应用都有官网地址，如果真没有那就填写应用的 Github 仓库地址吧，本质上就是一个联系地址

### github

应用的 Github 仓库地址，本质上提交到 1Panel 应用商店的应用都是开源应用，所以一般都有这么个开源地址，你本地定制可以按情况来。

### document

应用的使用文档地址，按需填写吧，不是所有应用都有完善的使用文档。

## 应用参数配置文件

它也是`data.yml`，但是它在应用的版本号目录下，注意不要和应用声明文件搞混了。

> 本文件主要用于生成安装时要填写的 form 表单，在应用版本文件夹下面 可以无表单，但是需要有这个 data.yml 文件，并且包含 formFields 字段。
> @1Panel 应用商店仓库官方 Wiki

以 Halo 的 Form 表单为例：

![Halo 安装表单示例图（来自 1Panel 官方应用商店仓库）](https://user-images.githubusercontent.com/31820853/226111412-9c7b25a1-83f2-4621-8789-7ef85a2695dd.png)
如果要生成一个上面这样子的表单，你需要这么填写应用参数配置文件（来自官方 Wiki 示例）：

```
additionalProperties:  #固定参数
    formFields:
        - default: ""
          envKey: PANEL_DB_HOST  #docker-compose 文件中的参数
          key: mysql  #依赖应用的 key , 例如 mysql
          labelEn: Database Service  #英文的 label
          labelZh: 数据库服务  #中文的 label
          required: true  #是否必填
          type: service  #如果需要依赖其他应用，例如数据库，使用此 type
        - default: halo
          envKey: PANEL_DB_NAME
          labelEn: Database
          labelZh: 数据库名
          random: true  #是否在 default 文字后面，增加随机字符串
          required: true
          rule: paramCommon  #校验规则
          type: text  #需要手动填写的，使用此 type
        - default: halo
          envKey: PANEL_DB_USER
          labelEn: User
          labelZh: 数据库用户
          random: true
          required: true
          rule: paramCommon
          type: text
        - default: halo
          envKey: PANEL_DB_USER_PASSWORD
          labelEn: Password
          labelZh: 数据库用户密码
          random: true
          required: true
          rule: paramComplexity
          type: password  #密码字段使用此 type
        - default: admin
          envKey: HALO_ADMIN
          labelEn: Admin Username
          labelZh: 超级管理员用户名
          required: true
          rule: paramCommon
          type: text
        - default: halo
          envKey: HALO_ADMIN_PASSWORD
          labelEn: Admin Password
          labelZh: 超级管理员密码
          random: true
          required: true
          rule: paramComplexity
          type: password
        - default: http://localhost:8080
          edit: true
          envKey: HALO_EXTERNAL_URL
          labelEn: External URL
          labelZh: 外部访问地址
          required: true
          rule: paramExtUrl
          type: text
        - default: 8080
          edit: true
          envKey: PANEL_APP_PORT_HTTP
          labelEn: Port
          labelZh: 端口
          required: true
          rule: paramPort
          type: number #端口使用此 type
```

是不是看的眼晕了，我们这里拆出来解释下它这个东西：

```
        - default: ""
          envKey: PANEL_DB_HOST  #docker-compose 文件中的参数
          key: mysql  #依赖应用的 key , 例如 mysql
          labelEn: Database Service  #英文的 label
          labelZh: 数据库服务  #中文的 label
          required: true  #是否必填
          type: service  #如果需要依赖其他应用，例如数据库，使用此 type
```

### dafault

应用这个变量值的默认格式，填写上什么应用的设置变量默认就会填写什么东西，比如说你要设置这个应用的默认前端站点是`example.com`就在这里这样填写上它，应用就会默认把`example.com`做前端默认站点值。

### envKey

对应了你的应用的 compose 编排文件中环境变量设置部分，比如`environment`下的值，但其实简单点说就是会传递一个变量值给文件，比如`HOST`变量可以在 compose 文件中通过`${HOST}`来引用它，不难理解吧。

譬如在如下 Halo 例子中的引用：

```
services:
  halo:
    image: halohub/halo:2.2.0
    container_name: ${CONTAINER_NAME}
    restart: always
    networks:
      - 1panel-network
    volumes:
      - ./data:/root/.halo2
    ports:
      - ${PANEL_APP_PORT_HTTP}:8090
    command:
      - --spring.r2dbc.url=r2dbc:pool:${HALO_PLATFORM}://${PANEL_DB_HOST}:${HALO_DB_PORT}/${PANEL_DB_NAME}
      - --spring.r2dbc.username=${PANEL_DB_USER}
      - --spring.r2dbc.password=${PANEL_DB_USER_PASSWORD}
      - --spring.sql.init.platform=${HALO_PLATFORM}
      - --halo.external-url=${HALO_EXTERNAL_URL}
      - --halo.security.initializer.superadminusername=${HALO_ADMIN}
      - --halo.security.initializer.superadminpassword=${HALO_ADMIN_PASSWORD}
    labels:
      createdBy: "Apps"

networks:
  1panel-network:
    external: true
```

### key

这个虽然出现在了 1Panel 的官方 Wiki 示例中，但是似乎实际他们已经弃用这样写了，部分应用会依赖其他服务，比如 Halo 会有依赖 MySQL 数据库，这个时候你就在这里填写上它就能去引用它，1Panel 给的这个例子有点旧，我们使用下面的格式来实现引用外部数据库：

```
        - child:
            default: ""
            envKey: PANEL_DB_HOST
            required: true
            type: service
          default: mysql
          envKey: PANEL_DB_TYPE
          labelEn: Database Service
          labelZh: 数据库服务
          required: true
          type: apps
          values:
            - label: MySQL
              value: mysql
            - label: MariaDB
              value: mariadb
```

这样你就可以为你的应用接入选择数据库的方式了，但是我也不是很懂这个，实际写起来直接 Ctrl+C 其他已实现接入外部数据库的应用范例似乎是个好办法。

### labelEn/Zh

这个我感觉并不需要过多的介绍，已经在上面写的比较明白了，应用这个项的中英文描述。

### required

决定这个变量值是否是必填的，在它为`true`的情况下不填写它是不能继续安装的，你可以视情况决定这个东西。

### type

看着似乎表格很明白但实际写起来还是挺头疼的东西，自己按表格查询吧，我自己也对这个东西非常头疼，差不多它的设定决定了你的应用这个设置变量只能填什么东西。

| 值 | 说明 |
| --- | --- |
| service | `type: service` 如果该应用需要依赖其他组件，如 mysql redis 等，可以通过 `key: mysql` 定义依赖的名称，在创建应用时会要求先创建依赖的应用。 |
| password | `type: password` 敏感信息，如密码相关的字段会默认不显示明文。 |
| text | `type: text` 一般内容，比如数据库名称，默认明文显示。 |
| number | `type: number` 一般用在端口相关的配置上，只允许输入数字。 |
| select | `type: select` 选项，比如 `true`, `false`，日志等级等。 |
| 通过官方给的下方的例子对照查询了解： |  |

```
# type: service，定义一个 mysql 的 service 依赖。
- default: ""
    envKey: DB_HOST
    key: mysql
    labelEn: Database Service
    labelZh: 数据库服务
    required: true
    type: service

# type: password
- default: Np2qgqtiUayA857GpuVI0Wtg
    edit: true
    envKey: DB_PASSWORD
    labelEn: Database password
    labelZh: 数据库密码
    required: true
    type: password

# type: text
- default: 192.168.100.100
    disabled: true.
    envKey: REDIS_HOST
    labelEn: Redis host
    labelZh: Redis 主机
    type: text

# type: number
- default: 3306
    disabled: true
    envKey: DB_PORT
    labelEn: Database port
    labelZh: 数据库端口
    rule: paramPort
    type: number

# type: select
- default: "ERROR"
    envKey: LOG_LEVEL
    labelEn: Log level
    labelZh: 日志级别
    required: true
    type: select
    values:
        - label: DEBUG
          value: "DEBUG"
        - label: INFO
          value: "INFO"
        - label: WARNING
          value: "WARNING"
        - label: ERROR
          value: "ERROR"
        - label: CRITICAL
          value: "CRITICAL"
```

### rule

在上面的例子中部分登场，主要是校验用户是否书写这个东西的格式是正确的，比如防止某些用户在端口设置这里填写域名的东西，这个`rule`就会强制这个填写栏必须填写什么格式。

| 值 | 规则 |
| --- | --- |
| paramPort | 用于限制端口范围为 1-65535 |
| paramExtUrl | 格式为 http(s)://(域名/ip):(端口) |
| paramCommon | 英文、数字、.-和\_，长度 2-30 |
| paramComplexity | 支持英文、数字、.%@$!&~\_-,长度 6-30，特殊字符不能在首尾 |

## 应用 Compose 文件

位于应用版本号目录下的`docker-compose.yml`文件，注意有格式要求，下面是示例

### 示例文件

```
services:
  ghost:
    container_name: ${CONTAINER_NAME}
    restart: always
    networks:
      - 1panel-network
    ports:
      - "${PANEL_APP_PORT_HTTP}:2368"
    volumes:
      - ./data:/var/lib/ghost/content
    environment:
      - database__client=${PANEL_DB_TYPE}
      - database__connection__host=${PANEL_DB_HOST}
      - database__connection__user=${PANEL_DB_USER}
      - database__connection__password=${PANEL_DB_USER_PASSWORD}
      - database__connection__database=${PANEL_DB_NAME}
      - database__connection__port=${PANEL_DB_PORT}
      - url=${GHOST_EXTERNAL_URL}
    image: ghost:5.96.2
    labels:
      createdBy: "Apps"

networks:
  1panel-network:
    external: true
```

### 要求

- 头部没有`version`的变量，记得把它删去。

- 必须为应用定义`container_name`这个设置，且后面必须是`${CONTAINER_NAME}`，这样 1Panel 才能在安装时对其正确配置。

- 必须配置应用网络为`1panel-network`，这样应用之间才能正常互联，我在部分第三方应用商店库发现它们不是很注重这个，当然这事也可以视情况而论，但是建议是都在 1Panel 官方网络下。

- 端口`ports`这里设置也是必须为其默认定义的`${PANEL_APP_PORT_HTTP}:8090`这样的格式，这样 1Panel 才能正确设置应用的开放端口

- 文件尾部也必须跟示例文件一样，填写了`networks`配置，直接`Ctrl+C`过去就好。

- 应用的`image`镜像标签必须和文件夹表示的版本号一致，比如这个版本号文件夹写的是`5.96.2`，那么其文件内部`image`标签也必须那样写，不然会影响应用检测和安装更新！

- 记得填写上面的`labels`标签，直接`Ctrl+C`过去就行，当然这个也不算必填项，只是一种规范罢了。

- 记得为应用定义重启相关，比如`always`等等，这样保证应用在遇到问题后可以恢复（雾）

## 脚本

### 介绍

1Panel 在 安装之前、升级之前、卸载之后支持执行 .sh 脚本
分别对应 init.sh upgrade.sh uninstall.sh
存放目录 (以 halo 为例) : halo/2.2.0/scripts

## 本地测试

我十分建议你发布前先在自己已安装 1Panel 的服务器上进行测试，确定它是否可以正常安装

将应用目录上传到 1Panel 的`/opt/1panel/resource/apps/local`文件夹下
注意：`/opt` 为 1Panel 默认安装目录，请根据自己的实际情况修改
上传完成后，目录结构如下：

```
├──halo
    ├── logo.png
    ├── data.yml
    ├── README.md
    ├── 2.2.0
     &nbsp;&nbsp; ├── data.yml
     &nbsp;&nbsp; ├── data
     &nbsp;&nbsp; └── docker-compose.yml
```

在 1Panel 应用商店中，点击更新应用列表按钮同步本地应用。

## 收尾

测试正常后，你就可以去提交官方库或者第三方应用商店库了，或者你自己用，都可以，不过我还是建议你本着分享精神，把它提交到官方或其他第三方应用商店仓库里去。

官方的要求和拒绝率比较高，一般来说提交第三方仓库也挺好的，这里列举目前个人已知仓库：

- [AuroraStarTeam/1panel-app-store](https://github.com/AuroraStarTeam/1panel-app-store)，由我所属个人组织极光星进行维护，理念是干净整洁，但是个人经验可能不够丰富。

- [okxlin/appstore](https://github.com/okxlin/appstore)，流行和最早的第三方商店库，应用很多，但是也很乱。

## 参考文档：

- [如何提交自己想要的应用（1Panel 官方应用商店仓库 Wiki）](https://github.com/1Panel-dev/appstore/wiki/%E5%A6%82%E4%BD%95%E6%8F%90%E4%BA%A4%E8%87%AA%E5%B7%B1%E6%83%B3%E8%A6%81%E7%9A%84%E5%BA%94%E7%94%A8)
