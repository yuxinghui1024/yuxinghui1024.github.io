---
title: EL7安装MYSQL
date: 2021-05-16 15:38:06
---

### mysql5.7安装
系统环境：RHEL7，不适用于Fedora 32, 33, and 34

**注意：** 如果安装过第三方版本请参考 [Replacing a Native Third-Party Distribution of MySQL.](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/index.html#repo-qg-yum-replacing)

#### 上传下载工具安装（非必须）：
也可以使用ftp工具上传下载，属于个人喜好
```shell
yum install lrzsz -y
```
#### 使用yum安装MYSQL
1. 添加MySQL Yum存储库
进入[下载地址](https://dev.mysql.com/downloads/repo/yum/)选择对应的rpm包，执行
```
shell> rpm -Uvh platform-and-version-specific-package-name.rpm
```
**注意：**通过yum update命令进行的所有系统范围内的更新都将自动升级系统上的MySQL软件包，并且还会替换任何本机的第三方软件包（如果有的话） 在MySQL Yum存储库中找到它们的替代品。

2. 选择存储库中的MYSQL版本安装
使用MySQL Yum存储库时，默认情况下会选择安装最新的GA版本的MySQL因为默认启动最新版本subrepositories ，所以需要手动指定安装版本。
可以使用命令查看当前启动的子存储库：
```shell
yum repolist all | grep mysql
```
修改使用的子存储库
```shell
shell> yum-config-manager --disable mysql80-community
shell> yum-config-manager --enable mysql57-community
```
或者可以手动修改配置文件`/etc/yum.repos.d/mysql-community.repo`
```shell
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/6/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```
检查是否生效
```shell
yum repolist all | grep mysql
```
**注意：**EL8包含MySQL module，它会屏蔽MYSQL的仓库，需要禁用`yum module disable mysql`

3. 安装mysql
```shell
shell> yum install mysql-community-server
```


4. 启动服务
```shell
shell> systemctl start mysqld
shell> systemctl status mysqld
```

**注意：**
从MySQL5.7之后，会默认安装[validate_password plugin](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html),且当数据目录是空时会做如下动作：
* 初始化服务器
* SSL证书和密钥文件在数据目录中生成。
* 创建一个超级用户帐户“ root” @“ localhost”。 设置超级用户的密码并将其存储在错误日志文件中。 要显示它，请使用以下命令：
```shell
shell> grep 'temporary password' /var/log/mysqld.log
```
通过临时密码修改root密码：
```shell
shell> mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```




