## firewall
修改配置后需要重载防火墙
```
firewall-cmd --state                      #查看状态
systemctl start firewalld                 #启动
systemctl enable firewalld                #开机自启动
firewall-cmd --reload                     #重载
firewall-cmd --get-default-zone           #查看默认区域
firewall-cmd --get-active-zones           #查看活跃区域
firewall-cmd --permanent --list-port      #查看已放行端口
firewall-cmd --zone=public --list-all
firewall-cmd --permanent --zone=public --add-port=22/tcp #永久放行 22/tcp 端口
```

## iptables
iptables 配置文件位置`/etc/sysconfig/iptables`
通过指令修改配置后需要保存规则然后重启服务
通过编辑 iptables 配置文件修改配置后，需要先重启服务，再保存规则
```
service iptables save         #保存规则
service iptables restart      #重启服务
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT #放行 22/tcp 端口
```
