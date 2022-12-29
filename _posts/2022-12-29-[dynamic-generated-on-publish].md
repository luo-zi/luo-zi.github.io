---
layout: post
date: 2021-12-06 01:09:31
categories: []
title: MySQL安装简明(清华镜像)
permalink: /post/mysql-installation-by-tsinghua-mirror.html
tagline: >-
  mysql安装简明(清华镜像)mysql安装１下载https_mirrorstunatsinghuaeducnmysqldownloadsmysqlmysqlwinxzip这是mysql的下载链接安装使用winrar等软件解压mysqlwinxzip(win本身也支持zip文件的解压)把解压出来的文件夹的路径加入到环境变量(path)中进入解压出来的文件夹新建一个txt文件(目录空白处右键新建文本文档)将以下内容复制进文件稍作修改注意看注释_[mysqld]#将at_at_at_替换为解压出来的文件夹的路
tags:
  - 解压
  - 安装
  - 文件
  - 下载
  - 出来
published: true
---

# MySQL安装简明(清华镜像)

## MySQL 安装

### １.下载

```
https://mirrors.tuna.tsinghua.edu.cn/mysql/downloads/MySQL-8.0/mysql-8.0.26-winx64.zip //这是MySQL8.0的下载链接
```

### 安装

1. 使用WinRAR等软件解压mysql-8.0.26-winx64.zip(win10本身也支持zip文件的解压)
2. 把解压出来的文件夹的路径加入到环境变量(Path)中
3. 进入解压出来的文件夹,新建一个txt文件(目录空白处右键-新建-文本文档),将以下内容复制进文件,稍作修改,注意看注释!

```bash
[mysqld]
# 将@@@替换为解压出来的文件夹的路径
basedir=@@@
# 将%%%替换为任意的目录,这个目录将用于存储数据
datadir=%%%

```

3. 将新建的txt文件改名为my.ini,
   注意,这里有同学常犯一个错误:
   没有开启文件后缀的显示,在这种情况下,修改出来文件名为my.ini.txt,
   特征是这种情况下的my.ini.txt文件可以被直接双击打开,
   而正确修改的my.ini文件通常不能直接双击打开(通常).

### 数据库服务端的初始化

1. 以管理员模式打开cmd(如果不会,请按win+x,点击Powershell(管理员),这个也是可以的),运行以下指令.

   ```bash
    mysqld --defaults-file=./my.ini --initialize --console
   ```
2. 观察输出的最后一行,将**冒号**后的**字符串复制并妥善保存**(这是数据库Root用户的密码)

   ### 数据库服务端的启动

   启动时,使用以下代码进行启动(同样可以保存为bat文件以便快速运行)

   ```bash
   mysqld --defaults-file=./my.ini --console
   ```

   正常启动后,观察程序输出的最后一行:

```bash
[Note] A temporary password is generated for root@localhost: 这里就是密码!!!

##如果你没看见这行,不好意思,初始化没成功,请清空datadir填的那个文件夹,然后看看本文末尾的疑难杂症系列,重新试一次.
```

## 使用 MySQL

### MySQL 客户端连接到服务端

1. 打开cmd(正常启动一个cmd即可,不要关闭服务端!!!新开一个cmd进行下面的操作!)
2. 输入以下指令:

   ```bash
   Mysql -h localhost -u root -p
   ```
3. 此时出现密码输入界面,将此前保存的密码复制或键入,按下Enter键
4. 如果此时弹出的文本内容类似下方,则为连接成功:

   ```text
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 8
   Server version: 8.0.26

   Copyright (c) 2000, 2021, Oracle and/or its affiliates.

   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

   mysql>
   ```
5. 在确认已经连接成功后,MySQL要求用户必须对Root的初始密码进行修改,请输入以下代码:

   ```sql
   ALTER USER 'root'@ 'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';
   #这条命令是为了让图形化的MySQL客户端能连上,8.0后版本有这个毛病,5.x版本不需要.(做作业用这条)

   ALTER USER USER() IDENTIFIED BY '你的密码';
   # 5.7.6版本之后使用这条命令修改

   #下面这条是5.7.6之前的版本可使用的修改密码指令
   SET PASSWORD=PASSWORD('这里是你的密码'); #别忘了分号

   FLUSH PRIVILEGES;#刷新数据库对密码的缓存.
   ```

   本步骤仅进行一次,即:第一次登录到root账户时
6. 尝试一点基础的命令吧!

   ```sql
   show databases;
   #展示当前数据库(mysql数据库)所拥有的数据表
   #这个数据库存储了mysql的用户信息和各类配置等信息
   ```

## 卸载后重装?

##　完整的卸载

1. 删除数据库安装目录下文件.默认路径为`C:\Program Files\MySQL`
2. 删除数据目录.  默认路径为`C:\ProgramData\MySQL`  ,如果是本文步骤安装的MySQL,应将datadir中填入的文件夹路径下的文件删除.
3. 尝试使用以下脚本删除残余的注册表项（可用同类工具代替）

```powershell
#保存为removeSQl.ps1
Remove-Item -Path Registry::HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\EventLog\Application\MySQLD Service
Remove-Item -Path Registry::HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\EventLog\Application\MySQLD Service
```

4. 按本文介绍的安装步骤安装即可.

## 各种疑难杂症(自查)

### cmd 工作目录和 my.ini 的所在目录不一致

常出现在这部分`mysqld --defaults-file=./my.ini`,这里使用的路径是相对路径,举个例子:

```bash
#很多人cmd打开默认就在这,此时./my.ini的意思是在这个目录下寻找my.ini
C:\Windows\system32>   #这里是你输入命令的那一行   
```

   那么如何解决这个问题?

#### 改为绝对路径(不推荐)

```bash
   mysqld --defaults-file=C:\MySQL\my.ini #后面接上其他参数,参考正文中初始化和启动的样例
   #这里从C盘开始寻址,完整地给出了一条路径,这就是绝对路径
```

   绝对路径法的缺点是需要根据具体情况更改路径,在目录中含有空格时(比如`C:\Program Files\MySQL`)需要用双引号包围:`"C:\Program Files\MySQL"`

#### 使用相对路径(推荐)

```bash
   mysqld --defaults-file=./my.ini #后接其他参数
   #按照cmd或powershell的当前工作目录,在此目录中寻找my.ini
```

   为了使用相对路径,你需要用cd命令进入到my.ini的所在目录(本教程中,my.ini与MySQL的目录放在一起,方便管理,使用)
   cd命令的使用方法如下:

```bash
   cd C:\Users #工作目录切换到此路径
   cd .. 
   # 退一层目录,若执行本行命令前工作目录为C:\Users,则执行后工作目录为C:\
```

   进入my.ini所在目录后运行命令,相对路径法将正常运行.

### 环境变量没有设置

   凡是在cmd中输入mysql或mysql提示不是程序或文件的,70%是这种情况
   MySQL目录下的bin文件夹,里面有mysql,mysqld等你需要用的命令的本体,你不把路径放进环境变量(Path变量)里,cmd怎么帮你找???

### datadir 乱写

   这个路径写了是要被数据库写一大堆数据文件进去的,自己开个文件夹,然后把这个文件夹的路径填进去,这个文件夹在初始化时是一定必须为空,里边一点东西都不能有!!

### 提示缺少 MSVC120.dll

下载并安装可解决:

```bash
https://www.microsoft.com/zh-cn/download/confirmation.aspx?id=40784
```

> 作者:曾建闻
> 勘误&提问邮箱:zjw1355528158@outlook.com
> 致谢:卢泳诗同学带来了好问题,陈熳和张宏博同学的耐心交流都使得这个文档越来越完善  ~~头发?头发已经无所谓了~~
> --->MySQL清晰的官方文档挽救了我的脑子.

> **引用**:
> [数据库的账号与密码设置](https://www.mysqlzh.com/doc/45/182.html)
> [数据库的初始化](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html#sa79741230)
> [使用注册表项 - PowerShell | Microsoft Docs](https://docs.microsoft.com/zh-cn/powershell/scripting/samples/working-with-registry-keys?view=powershell-5.1#listing-all-subkeys-of-a-registry-key)

‍
