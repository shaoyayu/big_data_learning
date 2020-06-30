# Keepalived

### 简介

 Keepalived的作用是检测服务器的状态，如果有一台web服务器宕机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。 

keepalived是以VRRP协议为实现基础的，VRRP全称Virtual Router Redundancy Protocol，即虚拟路由冗余协议。

虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master和多个backup，master上面有一个对外提供服务的vip（该路由器所在局域网内其他机器的默认路由为该vip），master会发组播，当backup收不到vrrp包时就认为master宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master。这样的话就可以保证路由器的高可用了。

- Keepalived是集群管理中保证集群高可用的服务软件
- 高可用 High Available
- 1、需要心跳机制探测后端RS是否提供服务
  - 探测down,需要从lvs中删除该rs
  - 探测发送从down到up，需要从lvs中再次添加RS
- LVS DR，需要主备(HA)

### 原理：

VRRP协议（虚拟路由冗余协议），IP漂移

### Keepalived安装配置

#### 安装

yum install keepalived

启动：service keepalived start

配置文件: /etc/keepalived/keepalived.conf

tail /var/log/message查看日志

 

### 实验：

/etc/keepalived/keepalived.comf下面的配置：

主服务上：

```shell
vrrp_instance VI_1 {
    state MASTER	#区分主备，当主服务器强修过来重置这个参数就可以直接抢回路由，
    interface eth0	#采集的数据包网卡
    virtual_router_id 51	#虚拟id
    priority 100	#谁大谁接替挂掉的主机
    advert_int 1	#
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress { #配置接受的网卡内容
        192.168.25.100/24 dev eth0 label eth0:2
    }
}
```

内核配置

```shell
virtual_server 192.168.25.100 80 {#配置拦截的包
    delay_loop 6
    lb_algo rr		#轮询模式
    lb_kind DR		#直接路由模型
    nat_mask 255.255.255.0
    persistence_timeout 50	#设置握手缓存的时间，在这段时间内都会负载到同一台服务器中，单位S
    protocol TCP
    
    real_server 192.168.201.100 443 {
        weight 1
        HTTP_GET {
            url {
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            connect_timeout 3	#健康检查参数
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

怎么说这个事情呢，，，反正学习很漫长，先来；

#### 主服务器配置文件；

```shell
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
	192.168.25.100/24 dev eth0 label eth0:2
    }
}

virtual_server 192.168.25.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    nat_mask 255.255.255.0
    persistence_timeout 0
    protocol TCP

    real_server 192.168.25.52 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200 
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.25.53 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

#### 次服务器

```shell
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
	192.168.25.100/24 dev eth0 label eth0:2
    }
}
virtual_server 192.168.25.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    nat_mask 255.255.255.0
    persistence_timeout 0
    protocol TCP

    real_server 192.168.25.52 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200 
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.25.53 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

#### 服务器相关配置：

```shell
[root@MDNode03 ~]# echo 1 > /proc/sys/net/ipv4/conf/eth0/arp_ignore
[root@MDNode03 ~]# echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
[root@MDNode03 ~]# echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
[root@MDNode03 ~]# echo 2 > /proc/sys/net/ipv4/conf/eth0/arp_announce 
[root@MDNode03 ~]# ifconfig lo:2 192.168.25.100 netmask 255.255.255.255
[root@MDNode03 ~]# ifconfig 
eth0      Link encap:Ethernet  HWaddr 00:0C:29:BA:98:84  
          inet addr:192.168.25.52  Bcast:192.168.25.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:feba:9884/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1292 errors:0 dropped:0 overruns:0 frame:0
          TX packets:834 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:112000 (109.3 KiB)  TX bytes:99000 (96.6 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

lo:2      Link encap:Local Loopback  
          inet addr:192.168.25.100  Mask:255.255.255.255
          UP LOOPBACK RUNNING  MTU:16436  Metric:1

[root@MDNode03 ~]# service httpd start
Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 192.168.25.52 for ServerName
                                                           [  OK  ]
```



#### 启动lvs服务器和备用lvs服务器

```shell
[root@MDNode01 keepalived]# service keepalived start
Starting keepalived:                                       [  OK  ]
```

##### 主服务

```shell
[root@MDNode01 keepalived]# ifconfig 
eth0      Link encap:Ethernet  HWaddr 00:0C:29:BF:3A:BE  
          inet addr:192.168.25.50  Bcast:192.168.25.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:febf:3abe/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3045 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4357 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:369272 (360.6 KiB)  TX bytes:347222 (339.0 KiB)

eth0:2    Link encap:Ethernet  HWaddr 00:0C:29:BF:3A:BE  
          inet addr:192.168.25.100  Bcast:0.0.0.0  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

```



##### 次服务器

```shell
eth0      Link encap:Ethernet  HWaddr 00:0C:29:A6:A5:1F  
          inet addr:192.168.25.51  Bcast:192.168.25.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fea6:a51f/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4205 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2637 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:417976 (408.1 KiB)  TX bytes:222544 (217.3 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

```



#### 主服务器和次服务器的内核ipvsadm

```shell
[root@MDNode01 keepalived]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.25.100:80 rr
  -> 192.168.25.52:80             Route   1      0          0         
  -> 192.168.25.53:80             Route   1      0          0    
```





