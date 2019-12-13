# Linux 命令

## CentOS 防火墙相关命令

1. 查看 firewall 服务状态

```shell
systemctl status firewalld
```



![systemctl-status-firewalld](E:\Code\Note\image\linux\systemctl-status-firewalld.jpg)

2. 查看 firewall 的状态

```shell
firewall-cmd --state
```

![](E:\Code\Note\image\linux\firewall-cmd-state.jpg)

3. 开启、重启、关闭 firewalld.service 服务

```shell
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
```

4. 查看防火墙规则

```shell
firewall-cmd --list-all 
```

![](E:\Code\Note\image\linux\firewall-cmd--list-all.jpg)

5. 查询、开放、关闭端口

```
# 查询端口是否开放firewall-cmd --query-port=8080/tcp# 开放80端口firewall-cmd --permanent --add-port=80/tcp# 移除端口firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙(修改配置后要重启防火墙)firewall-cmd --reload
# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```