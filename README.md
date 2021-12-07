# Zabbix 一键部署

**脚本适用于操作系统：CentOS7.X/RedHat7.X部署，内存1G内存，磁盘容量大于5G,部署环境最好使用单独的服务器，避免环境冲突。**

#### 环境介绍：

```
Linux localhost.localdomain 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
5.5.68-MariaDB
Apache/2.4.6 (CentOS)
PHP 7.0.33
```

#### 执行脚本：

```shell
#!/bin/sh
#
#Date:2021.10.27
#Author:GZ
#Mail:V2board@qq.com

process()
{
install_date="zabbix_install_$(date +%Y-%m-%d_%H:%M:%S).log"
printf "
\033[36m#######################################################################
#                     欢迎使用Zabbix一键部署脚本                      #
#                脚本适配环境CentOS7+/RetHot7+、内存1G+               #
#                更多信息请访问 https://gz1903.github.io              #
#######################################################################\033[0m
"

while :; do echo
    read -p "请输入Mysql数据库root密码: " Database_Password 
    [ -n "$Database_Password" ] && break
done

echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                    正在安装常用组件 请稍等~                         #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
yum clean all
yum install -y vim wget tar
echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                  正在关闭SElinux策略 请稍等~                        #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
setenforce 0
#临时关闭SElinux
sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
#永久关闭SElinux

echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                  正在配置Firewall策略 请稍等~                       #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=10050/tcp --permanent
firewall-cmd --zone=public --add-port=10051/tcp --permanent
 
firewall-cmd --reload
firewall-cmd --zone=public --list-ports
#放行TCP80、10050、10051端口
echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                 正在安装Mariadb数据库 请稍等~                       #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
yum install -y mariadb-server mariadb 
systemctl start mariadb
systemctl enable mariadb
echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#         正在安装配置PHP环境及扩展  时间较长请稍等~                  #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum install -y  php70w* --skip-broken
systemctl start php-fpm
systemctl enable php-fpm
echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                   正在配置Apache服务 请稍等~                        #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
systemctl start httpd
systemctl enable httpd
echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                   正在创建Zabbix用户 请稍等~                        #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
groupadd zabbix
useradd zabbix -g zabbix -s /sbin/nologin
echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                   正在编译Zabbix软件 请稍等~                        #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
yum install -y gcc  libxml2-devel libevent-devel net-snmp net-snmp-devel  curl  curl-devel php  php-bcmath  php-mbstring mariadb mariadb-devel java-1.6.0-openjdk-devel --skip-broken
#安装Zabbix编译的软件包
 
#去官网下载编译安装的Zabbix：https://www.zabbix.com/download_sources
wget https://www.xxshell.com/download/sh/zabbix/zabbix4.4/zabbix-4.4.1.tar.gz
tar -xzvf zabbix-4.4.1.tar.gz
cd zabbix-4.4.1

./configure  \
        --prefix=/usr/local/zabbix  \
        --enable-server  \
        --enable-agent  \
        --with-mysql=/usr/bin/mysql_config   \
        --with-net-snmp  \
        --with-libcurl  \
        --with-libxml2  \
        --enable-java  
 
make -j 2 && make install 
#编译安装Zabbix
echo $?="Zabbix编译完成"

echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                   正在配置Mariadb数据库 请稍等~                     #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
mysqladmin -u root password "$Database_Password"
echo "---mysqladmin -u root password "$Database_Password""
#修改数据库密码

mysql -uroot -p$Database_Password -e "CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_general_ci;"
echo $?="正在创建zabbix数据库"
#将创建数据的命令重定向到数据库中

mysql -uroot -p$Database_Password -e "use zabbix;"
echo $?="对zabbix数据库进行操作"
#将选中的命令重定向到数据库中

mysql -uroot -p$Database_Password  zabbix < database/mysql/schema.sql
mysql -uroot -p$Database_Password  zabbix < database/mysql/images.sql
mysql -uroot -p$Database_Password  zabbix < database/mysql/data.sql
echo $?="对zabbix数据库进行操作"
#zabbix数据库导入

echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                    正在配置Zabbix软件 请稍等~                       #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
cp misc/init.d/fedora/core/* /etc/init.d/
#拷贝启动文件到/etc/init.d/下
echo $?="拷贝启动文件到/etc/init.d/下"

sed -i "s#BASEDIR=/usr/local#BASEDIR=/usr/local/zabbix#" /etc/init.d/zabbix_server
sed -i "s#BASEDIR=/usr/local#BASEDIR=/usr/local/zabbix#" /etc/init.d/zabbix_agentd
#编辑启动模块下
echo $?="编辑启动模块"

sed -i "s|# DBHost=localhost|DBHost=localhost|" /usr/local/zabbix/etc/zabbix_server.conf
sed -i "s|DBUser=zabbix|DBUser=root|" /usr/local/zabbix/etc/zabbix_server.conf
sed -i "s|# DBPassword=|DBPassword=$Database_Password|" /usr/local/zabbix/etc/zabbix_server.conf
#编辑Zabbix配置配置文件
echo $?="编辑Zabbix配置配置文件"

/etc/init.d/zabbix_server restart
/etc/init.d/zabbix_agentd restart
#启动zabbix服务
 
systemctl restart zabbix_server 
systemctl restart zabbix_agentd
#重启验证服务
#通过”netstat -an | grep LIS“查看10050、10051端口能否正常监听，如果不能正常监听可能数据库或配置文件有问题。
 
systemctl enable zabbix_server
systemctl enable zabbix_agentd
echo $?="配置Zabbix完成"
echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                    正在配置PHP.ini 请稍等~                          #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"
sed -i "s/post_max_size = 8M/post_max_size = 32M/" /etc/php.ini
sed -i "s/max_execution_time = 30/max_execution_time = 600/" /etc/php.ini
sed -i "s/max_input_time = 60/max_input_time = 600/" /etc/php.ini
sed -i "s#;date.timezone =#date.timezone = Asia/Shanghai#" /etc/php.ini
#修改PHP配置文件
echo $?="PHP.inin配置完成完成"

echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#               正在配置Zabbix前台文件 请稍等~                        #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"

rm -rf /var/www/html/*
#清空网站根目录
cp -r frontends/php/* /var/www/html/
#复制PHP文件到网站根目录
chown -R apache:apache  /var/www/html/
chmod -R 777 /var/www/html/conf/
#给网站目录添加属主
# 修复中文乱码
yum install wqy-microhei-fonts -y
yes | cp -i /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /var/www/html/assets/fonts/DejaVuSans.ttf
echo $?="网页文件拷贝完成"

#获取主机内网ip
ip="$(ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}')"
#获取主机外网ip
ips="$(curl ip.sb)"


echo -e "\033[36m#######################################################################\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#                       正在重启服务 请稍等~                          #\033[0m"
echo -e "\033[36m#                                                                     #\033[0m"
echo -e "\033[36m#######################################################################\033[0m"

systemctl restart php-fpm httpd mariadb zabbix_server zabbix_agentd
echo $?="服务启动完成"

echo -e "\033[32m--------------------------- 安装已完成 ---------------------------\033[0m"
echo -e "\033[32m 数据库名       :zabbix\033[0m"
echo -e "\033[32m 数据库用户名   :root\033[0m"
echo -e "\033[32m 数据库密码     :"$Database_Password
echo -e "\033[32m 网站目录       :/var/www/html \033[0m"
echo -e "\033[32m 配置文件目录   :/usr/local/zabbix/etc/zabbix_server.conf \033[0m"
echo -e "\033[32m Zabbix内网访问 :http://"$ip
echo -e "\033[32m Zabbix外网访问 :http://"$ips
echo -e "\033[32m 安装日志文件   :/var/log/"$install_date
echo -e "\033[32m------------------------------------------------------------------\033[0m"
echo -e "\033[32m 如果安装有问题请反馈安装日志文件。\033[0m"
echo -e "\033[32m 使用有问题请在这里寻求帮助:https://gz1903.github.io\033[0m"
echo -e "\033[32m 电子邮箱:v2board@qq.com\033[0m"
echo -e "\033[32m------------------------------------------------------------------\033[0m"
}
LOGFILE=/var/log/"zabbix_install_$(date +%Y-%m-%d_%H:%M:%S).log"
touch $LOGFILE
tail -f $LOGFILE &
pid=$!
exec 3>&1
exec 4>&2
exec &>$LOGFILE
process
ret=$?
exec 1>&3 3>&-
exec 2>&4 4>&-
```

![sh](https://cdn.jsdelivr.net/gh/gz1903/tu/8d38221f05cd7be77ca3901fc5a95a09.png)

![ok](https://cdn.jsdelivr.net/gh/gz1903/tu/7311caab63658ad8b8a3599e8e867a4c.png)
