## 后台开启代理为什么不能连接到校园网？

​	这是 Linux 校园网用户最经典的“代理与内网打架”问题。

​	校园网的认证页面（Captive Portal）通常是一个局域网 IP（比如 `10.x.x.x`）或者一个特定的内网域名。

​	当你打开 clash-verge 时，它接管了系统的网络流量。校园网的认证请求被 clash 拦截，当成普通网页发送给了你的远端代理节点。代理节点显然访问不到你学校的内网服务器，所以页面根本弹不出来。

​	很多校园网登录后，会在后台定期发送“心跳包”或者需要重新解析 DNS 来维持登录状态。如果你的 clash 处于全局接管状态（尤其是开启了 TUN 模式或 Fake-IP），这些维持网络的本地请求又被代理带跑了，校园网网关收不到你的心跳包，就会判定你离线，把你踢下线。



> 这是个很麻烦的问题, 这意味着你需要过一会就退出代理软件重新连接校园网，然后再开启代理软件。



## 解决办法

​	先关掉 clash，连上校园网，正常跳转到登录页面。 看一眼浏览器的地址栏，记下那个**域名**（比如 `login.xxx.edu.cn`）或者 **IP 地址**（比如 `10.1.2.3`）。



如果使用的是 **System Proxy（系统代理）** 模式：

1. 打开 clash-verge 的 **Settings（设置）**。
2. 找到 **System Proxy（系统代理）** 设置项。
3. 里面会有一个 **Bypass Domain/IP（绕过域名/IP）** 的列表。
4. 把你刚才记下的校园网域名或 IP 加进去。
   - 如果是 IP，建议把你学校所在的整个网段加进去，比如 `10.0.0.0/8` 或 `172.16.0.0/12`。
   - 如果是域名，直接加上去，比如 `login.xxx.edu.cn`。

![](/blogs/0302/a2936a47b012fb04.png)


如果你使用的是 **TUN 模式**，则需要编辑配置。

在 clash-verge 中找到 **Profile（配置）** -> 右键你当前使用的订阅 -> **Edit Rules（编辑规则）** 或者在全局扩展配置中修改。在 `rules` 列表的最前面加上：

```
- DOMAIN-SUFFIX,你学校的登录域名.edu.cn,DIRECT
- IP-CIDR,你学校的登录IP网段/24,DIRECT
```

![](/blogs/0302/accb5aac609643c4.png)


如果你在 clash-verge 中启用了 **Fake-IP** 模式，Clash 会返回一个伪造的 IP 给你的电脑，这会导致校园网的本地 DNS 劫持失效。

**解决方法：** 在 clash 的配置中，找到 `dns` -> `fake-ip-filter` 列表，把校园网的登录域名加进去：

```
*.你学校的域名.edu.cn, login.xxx.edu.cn
```

![](/blogs/0302/2edae1808ec92404.png)

设置完之后，尝试在clash开启的情况下访问登陆认证页面，如果访问成功，则没问题，如果没成功，就修改/etc/hosts，将ip映射为对应域名。



## 解决校园网经常断掉需要重连的问题

有的校园网，可能会不定时断掉，让你重新登录认证

> 这可能导致你在学习过程中，需要重新用浏览器进行登陆验证，非常繁琐。

第一步：抓取校园网的登录命令

​	先把校园网断开（或者注销当前登录状态），确保你现在处于需要认证的状态。

​	打开浏览器（比如 Chrome/Edge/Firefox），按 `F12` 打开**开发者工具**，切换到 **Network (网络)** 面板。

​	勾选面板上的 **Preserve log (保留日志)**（防止页面跳转后请求记录消失）。

​	在地址栏输入 `http://auth.xxx.edu.cn`(你学习登陆的网址)，像平时一样输入账号密码并点击登录。

​	登录成功后，在 Network 面板里找到那个**真正发送账号密码的请求**（通常是第一个或者第二个，方法一般是 `POST`，名字可能是 `login`、`auth` 或者一串字母）。

​	右键点击那个请求 -> **Copy (复制)** -> **Copy as cURL (bash)** 或 **以 cURL 格式复制**。


![](/blogs/0302/14bb70696d47c538.png)


第二步：编写自动监控与登录脚本

打开终端，在你喜欢的地方新建一个脚本文件，把下面的代码粘贴进去（修改需要修改的部分）：

```bash
#!/bin/bash

# ================= 配置区 =================
USER_ACCOUNT="" # 你的学号
USER_PASSWORD="" # 密码
# ==========================================

echo "[$(date '+%H:%M:%S')] 校园网自动登录脚本已启动！"

while true; do
    # 增加 -m 5 (最大执行时间5秒)，防止 curl 被死链卡住无响应
    if ! curl -s -I -m 5 http://connectivitycheck.platform.hicloud.com/generate_204 | grep -q "204"; then
        echo "[$(date '+%H:%M:%S')] ⚠️ 检测到断网或未认证，准备登录..."
        
        LOCAL_IP=$(ip -4 addr show | grep inet | grep -v '127.0.0.1' | grep -v '198.18' | awk '{print $2}' | cut -d/ -f1 | head -n 1)
        
        if [ -n "$LOCAL_IP" ]; then
            echo "[$(date '+%H:%M:%S')] 获取本机 IP: $LOCAL_IP"
            
            LOGIN_URL="http://你学校的登陆接口（可能跟我的不一样）/login?callback=dr1004&login_method=1&user_account=${USER_ACCOUNT}&user_password=${USER_PASSWORD}&wlan_user_ip=${LOCAL_IP}&wlan_user_ipv6=&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2.2&terminal_type=1&lang=zh-cn&v=3241&lang=zh" # 你的可能跟我的不一样，粘贴你的就行
            
            curl -s -m 5 "$LOGIN_URL" > /dev/null
            echo "[$(date '+%H:%M:%S')] ✅ 登录请求已发送！"
        else
            echo "[$(date '+%H:%M:%S')] ❌ 错误：无法获取本机 IP，Wi-Fi 断开了吗？"
        fi
        
        sleep 5
    else
        # 加上这行，让你知道它还活着
        echo "[$(date '+%H:%M:%S')] 🌐 网络畅通，无需登录。休眠 30 秒..."
    fi
    
    sleep 30
done
```

保存并退出。然后赋予这个脚本可执行权限：

```bash
chmod +x autologin.sh
```

你可以现在手动在终端里运行一下 `./autologin.sh` 测试看看效果，如果连着网它就什么都不做，如果没网它就会帮你静默发送登录请求。

![](/blogs/0302/749c9419891fe431.png)


> 如果成功帮你登陆了校园网，你可以把它注册成系统服务，然后开机自启它，这样就可以自动化连接你的校园网了。
