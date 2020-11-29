# CentOS 7代理设置(Yum/cURL/Wget/Docker)

很多企业员工不能直接访问Internet，通常需要通过Proxy访问，而且一般都需要使用员工账号和密码登录。

## 全局配置Proxy

全局配置Proxy时，对Yum，cURL，Wget同时生效。

### 命令行

代理配置直接在命令行中，这样的配置只在当前会话中生效，该会话断开后，该配置将不存在。

格式：

    export http_proxy=http://[username]:[password]@yourproxy:port/
    export https_proxy=http://[username]:[password]@yourproxy:port/

举例：

    export http_proxy=http://zhangsan:123456@10.112.2.2:8080/
    export https_proxy=http://zhangsan:123456@10.112.2.2:8080/

如果有些地址或者域名不需要代理，如公司内部地址，需要配置no_proxy。

    export no_proxy="127.0.0.1,localhost,server.example.com,192.168.1.2"

如配置192.168.50.10至192.168.50.100免代理。

    no_proxy_50=$(echo 192.168.50.{10..100})
    export no_proxy="127.0.0.1,localhost,${no_proxy_50}"

如配置192.168.50.*至192.168.51.*网段中10...100免代理。

    no_proxy_5051=$(echo 192.168.{50..51}.{10..100})
    export no_proxy="127.0.0.1,localhost,${no_proxy_5051}"
或

    printf -v no_proxy_50 '%s' 192.168.50.{10..100}
    export no_proxy="127.0.0.1,localhost,${no_proxy_50}"

如果密码中存在特殊字符的话，一般建议改为非特殊字符，也可以将特殊字符以十六进制的ASCII码表示（URL编码），格式为%HEX（忽略0x），部分常见特殊字符的ASCII码为：

| 字符 | ASCII码 |
| :-: | :-: |
| ! | 0x21 |
| # | 0x23 |
| $ | 0x24 |
| % | 0x25 |
| & | 0x26 |
| * | 0x2A |
| ? | 0x3F |
| @ | 0x40 |
| ^ | 0x5E |
| ~ | 0x7E |

举例，如果密码为`123456!`。

    $ export http_proxy=http://zhangsan:123456%21@10.112.2.2:8080/

### /etc/profile

如果需要永久生效，可以将相关配置设置在该用户的/etc/profile文件中，

    vi /etc/profile
    ......

    export http_proxy=http://zhangsan:123456@10.112.2.2:8080/
    export https_proxy=http://zhangsan:123456@10.112.2.2:8080/

    no_proxy_50=$(echo 192.168.50.{10..100})
    export no_proxy="127.0.0.1,localhost,${no_proxy_50}"

在`/etc/profile`中的配置在新建该用户会话时会自动生效，如果想在当前会话立即生效，可以执行如下命令。

    source /etc/profile

## Yum配置代理

如果需要对Yum单独配置代理，可以在Yum配置文件中设置代理，该代理立即永久生效。

    vi /etc/yum.conf
    proxy=http://10.112.2.2:8080
    proxy_username=zhangsan
    proxy_password=123456

## Wget配置代理

可以在~/.wgetrc文件配置，可以配置用户名和密码。

    vi /root/.wgetrc
    http_proxy = http://proxy.server.com:8080/
    https_proxy = http://proxy.server.com:8080/
    ftp_proxy = http://proxy.server.com:8080/
    --proxy-user=username
    --proxy-passwd=passwd

或者在/etc/wgetrc文件中配置，没有用户名和密码。

    vi /etc/wgetrc
    ......

    # You can set the default proxies for Wget to use for http, https, and ftp.
    # They will override the value in the environment.
    https_proxy = http://proxy.yoyodyne.com:18023/
    http_proxy = http://proxy.yoyodyne.com:18023/
    ftp_proxy = http://proxy.yoyodyne.com:18023/
    
    # If you do not want to use proxy at all, set this to off.
    use_proxy = on

    ......

也可以在命令行中直接配置代理。

    curl -x yourproxy:8080 http://www.get.this/
    curl -x yourproxy:8080 ftp://ftp.leachsite.com/README

    curl -u user:passwd -x yourproxy:8080 http://www.get.this/

    curl --noproxy localhost,get.this -x yourproxy:8080 http://www.get.this/

## cURL配置代理

    # 指定http代理IP和端口
    curl -x 113.185.19.192:80 http://www.get.this/
    curl --proxy 113.185.19.192:80 http://www.get.this/
     
    #指定为http代理
    curl -x http_proxy://113.185.19.192:80 http://www.get.this/
     
    #指定为https代理
    curl -x HTTPS_PROXY://113.185.19.192:80 http://www.get.this/
     
    #指定代理用户名和密码，basic认证方式
    curl -x username:password@113.185.19.192:80 http://www.get.this/
    curl -x 113.185.19.192:80 -U username:password http://www.get.this/
    curl -x 113.185.19.192:80 --proxy-user username:password http://www.get.this/
     
    #指定代理用户名和密码，ntlm认证方式
    curl -x 113.185.19.192:80 -U username:password --proxy-ntlm http://www.get.this/
     
    #指定代理协议、用户名和密码，basic认证方式
    curl -x http_proxy://username:password@113.185.19.192:80 http://www.get.this/

## Docker代理

需要创建目录和配置文件。

    mkdir /etc/systemd/system/docker.service.d
    vi /etc/systemd/system/docker.service.d/http-proxy.conf

如果需要用户和密码登录：

    [Service]
    Environment="HTTP_PROXY=http://[username]:[password]@yourproxy:port/" "HTTPS_PROXY=https://[username]:[password]@yourproxy:port/" "NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,192.168.12.11"

NO_PROXY列出了哪些Docker仓库不需要代理，一般公司内部的私有仓库是无需代理的。

如果不需要用户和密码登录:

    [Service]
    Environment="HTTP_PROXY=http://yourproxy:port/" "HTTPS_PROXY=https://yourproxy:port/" "NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,192.168.12.11"

具体例子：

    vi /etc/systemd/system/docker.service.d/http-proxy.conf
    [Service]
    Environment="HTTP_PROXY=http://zhangsan:123456@10.112.2.2:8080/" "HTTPS_PROXY=https://zhangsan:123456@10.112.2.2:8080/" "NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,192.168.12.11"

生效修改的服务配置，然后重启Docker服务。

    systemctl daemon-reload
    systemctl restart docker
