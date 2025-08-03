---
type: source-note
title: Nginx + Lua + Redis 实现动态 IP 黑名单
id: 20250803100827
created: 2025-08-03T10:08:27
source:
  - web
url: https://mp.weixin.qq.com/s/dYfXjeYO5BfQGTUT5TRbIg
tags:
  - source-note
  - redis
  - solution
processed: false
archived: false
---
Original Darius *2025年06月17日 10:08*

## 一、背景

为了有效封禁某些爬虫或恶意用户对服务器的请求，我们需要建立一个动态的 IP 黑名单机制。对于被列入黑名单的 IP 地址，我们将拒绝为其提供服务。

## 二、架构选择

实现 IP 黑名单的功能有多种途径：

1. **操作系统层面** ：通过配置 `iptables` 来拒绝指定 IP 的网络请求；
2. **Web Server 层面** ：利用 Nginx 自身的 `deny` 指令或者 Lua 插件来配置 IP 黑名单；
3. **应用层面** ：在请求服务之前检查客户端 IP 是否在黑名单中。

为了便于管理和共享，我们选择了 **Nginx + Lua + Redis** 的架构来实现 IP 黑名单功能。架构图如下：

![Image](https://mmbiz.qpic.cn/mmbiz_jpg/pwRh3D4UvxoLxAlXpnE9woJZLdLW2L2HCg7HKFn1ibuJQYEDiax8icqMoSSEIIdWZg2npVYR9G7xxsYNpeOQA72Lw/640?wx_fmt=jpeg&from=appmsg&randomid=xaytz7xs&watermark=1&tp=webp&wxfrom=5&wx_lazy=1)

  

## 三、实现步骤

### 1\. 安装 Nginx + Lua 模块

推荐使用 OpenResty，这是一个集成了各种 Lua 模块的 Nginx 服务器发行版，方便快速部署 Lua 支持。

### 2\. 安装并启动 Redis 服务器

确保 Redis 已安装并正常运行。本次通过 Docker 快速启动 Redis：

```
docker run -itd --name redis -p 6379:6379 redis
```

然后在 Redis 中创建一个集合用于存储黑名单 IP：

```
redis-cli
SADD ip_blacklist "192.168.56.1"
```

### 3\. 配置 Nginx

#### 3.1 修改 nginx.conf

在 Nginx 配置文件中添加以下内容，分配一块共享内存空间用于缓存 IP 黑名单，并指定 Lua 脚本位置：

```
http {
    # 分配 1M 共享内存用于存储 IP 黑名单
    lua_shared_dict ip_blacklist 1m;

    server {
        listen 80;
        server_name localhost;

        location = /ipblacklist {
            access_by_lua_file lua/ip_blacklist.lua;
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, lua</p>")
            ';
        }
    }
}
```

### 4\. 编写 Lua 脚本

将以下 Lua 脚本保存为 `ip_blacklist.lua` ，该脚本会定期从 Redis 获取最新的黑名单数据，并更新本地缓存。

```
-- Redis服务器地址
local redis_host = "your.redis.server.here"
-- Redis服务器端口
local redis_port = 6379

-- Redis连接超时时间（毫秒），不要设置得太高！
local redis_connect_timeout = 100

-- 要检查的黑名单集合的键名
local redis_key = "ip_blacklist"

-- 缓存查找的有效时间（秒）
local cache_ttl = 60

-- 结束配置部分

-- 获取客户端IP地址
local ip = ngx.var.remote_addr
-- 获取共享内存中的ip_blacklist
local ip_blacklist = ngx.shared.ip_blacklist
-- 获取上次更新的时间戳
local last_update_time = ip_blacklist:get("last_update_time")

-- 只有在cache_ttl秒之后才从Redis更新ip_blacklist：
if last_update_time == nilor last_update_time < (ngx.now() - cache_ttl) then

    -- 引入redis模块
    local redis = require"resty.redis";
    local red = redis:new();

    -- 设置Redis连接超时时间
    red:set_timeout(redis_connect_timeout);

    -- 尝试连接到Redis
    local ok, err = red:connect(redis_host, redis_port);
    ifnot ok then
        -- 如果连接失败，记录调试日志
        ngx.log(ngx.DEBUG, "Redis connection error while retrieving ip_blacklist: " .. err);
    else
        -- 从Redis获取新的ip_blacklist数据
        local new_ip_blacklist, err = red:smembers(redis_key);
        if err then
            -- 如果读取失败，记录调试日志
            ngx.log(ngx.DEBUG, "Redis read error while retrieving ip_blacklist: " .. err);
        else
            -- 替换本地存储的ip_blacklist为最新的值：
            ip_blacklist:flush_all();
            for index, banned_ip inipairs(new_ip_blacklist) do
                ip_blacklist:set(banned_ip, true);
            end
            -- 更新时间戳
            ip_blacklist:set("last_update_time", ngx.now());
        end
    end
end

-- 检查客户端IP是否在黑名单中
if ip_blacklist:get(ip) then
    -- 如果在黑名单中，记录调试日志并拒绝访问
    ngx.log(ngx.DEBUG, "Banned IP detected and refused access: " .. ip);
    return ngx.exit(ngx.HTTP_FORBIDDEN);
end
```

### 5\. 测试与生效

完成以上步骤后，重新加载 Nginx 配置以使更改生效：

```
nginx -s reload
```

此时，如果访问者的 IP 在黑名单中，将被拒绝访问，并返回 `403 Forbidden` 。

## 四、总结

通过上述方法，我们实现了基于 **Nginx + Lua + Redis** 的 IP 黑名单功能，具有以下优点：

1. **轻量高效** ：配置简单，几乎不增加服务器性能负担；
2. **集中管理** ：多台服务器可以通过同一个 Redis 实例共享黑名单数据；
3. **动态更新** ：可以手动或通过自动化方式更新 Redis 中的黑名单，无需重启服务即可生效。

这种方案非常适合用于需要动态控制访问权限的场景，如反爬虫、安全防护等。

修改于 2025年06月17日

继续滑动看下一个

TechOps之窗

向上滑动看下一个