# CentOS 7修改IP地址和主机名

修改为静态IP地址，以root用户操作。

    vi /etc/sysconfig/network-scripts/ifcfg-ens33
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    BOOTPROTO=static
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    NAME=ens33
    UUID=24705db8-d3e5-4d19-b4d6-d3f21bdf10e6
    DEVICE=ens33
    ONBOOT=yes
    IPADDR=192.168.1.55
    NETMASK=255.255.255.0
    GATEWAY=192.168.1.1
    DNS1=192.168.1.1
    DNS2=8.8.8.8

重启网络服务，使以上修改的静态IP地址生效。

    systemctl restart network

查看IP地址，第一个为回环地址，第二个为物理网卡地址。

    ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:0c:2e:23:88:1d brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.79/24 brd 192.168.1.255 scope global noprefixroute ens33
           valid_lft forever preferred_lft forever
        inet6 fe80::20c:29ff:ee4f:881d/64 scope link
           valid_lft forever preferred_lft forever

修改主机名。

    hostnamectl set-hostname k8s-master

查看主机名。

    hostname
    k8s-master
