---
title: Linux常用指令合集
date: 2021-08-12 23:49:59
tags: 
- Linux常用指令
categories: 
- Linux
description: 
sticky: 2
cover: https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/undraw_Coding_re_iv62.png
---

## Vi / Vim快捷键

- **拷贝：**`yy `, 拷贝当前行向下的 5 行 `5yy`

- **粘贴：**`p`

- **删除：**`dd`，删除当前行向下的 5 行` 5dd`
- **撤销：**`u`
- **查找：**`/关键字` ， 回车 查找 , n 就是查找下一个

- **设置行号：**`:set nu`
- **取消行号：**`:set nonu`
- **快速定位：**最首行【`gg`】   最末行【`G`】
- **光标移动：**输入 20,再输入 shift+g  即可定位到20行

- **保存退出：**`:wq`
- **不保存退出：**`:q`
- **不保存强制退出：**`:q!`

## 开机、重启和用户登录注销

- **关机：**
  - `shutdown -h now`
  - `halt`

- **一分钟后关机：**`shutdown -h 1`
- **重启：**
  - `shutdown -r now`
  - `reboot`

- **数据同步：**`sync`

> - 不管是重启系统还是关闭系统，首先要运行 **sync** 命令，把内存中的数据写到磁盘中
> - 目前的 shutdown/reboot/halt 等命令均已经在关机前进行了 sync ，但是**小心驶得万年船**

- **注销用户：**`logout`

> logout 注销指令在图形运行级别无效，在运行级别 3 下有效

## 用户管理

- **添加用户：**
  - `useradd 用户名`
  - `useradd -d 指定目录 用户名`

> 当创建用户成功后，会自动的创建和用户同名的家目录，通过`useradd -d `可以给新创建的用户指定家目录

- **指定/修改密码：**`passwd 用户名`

- **删除用户：**
  - `userdel 用户名`
  - `userdel -r 用户名`

> `userdel`删除用户但是**保留**家目录，`userdel -r`删除用户**不保留**家目录
>
> **一般情况下，我们建议保留家目录**

- **查看用户：**`id 用户名`

> 当用户不存在时，返回无此用户

- **切换用户：**`su - 用户名`

> 从权限高的用户切换到权限低的用户，不需要输入密码，反之需要
>
> 当需要返回到原来用户时，使用 `exit/logout` 指令

- **查看当前用户：**`whoami/ who am I`

- **新增组：**`groupadd 组名`

- **删除组：**`groupdel 组名`

- **给用户分配组：**`useradd –g 用户组 用户名`

- **修改用户的组：**`usermod –g 用户组 用户名`

> - `/etc/passwd` 文件：用户（user）的配置文件，记录用户的各种信息
>
> 每行的含义：**用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录 Shell**
>
> - `/etc/shadow` 文件：口令的配置文件
>
> 每行的含义：**登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志**
>
> - `/etc/group` 文件：组(group)的配置文件，记录 Linux 包含的组的信息
>
> 每行含义：**组名:口令:组标识号:组内用户列表**

## 实用指令

### 运行级别

- **切换运行级别：**`init [0,1,2,3,4,5,6]`

> 0  ：关机
>
> 1  ：单用户【找回丢失密码】
>
> 2：多用户状态没有网络服务
>
> 3：多用户状态有网络服务
>
> 4：系统未使用保留给用户
>
> 5：图形界面
>
> 6：系统重启
>
> 常用运行级别是 3 和 5 ，也可以指定默认运行级别

- **查看当前默认运行级别：**`systemctl get-default`

- **设置默认运行级别：**
  - **3的运行级别：**`systemctl set-default multi-user.target`
  - **5的运行级别：**`systemctl set-default graphical.target`

### 帮助指令

#### man 获得帮助信息

- **基本语法**：`man [命令或配置文件]`
- **功能描述**：获得帮助信息

> 查看 ls 命令的帮助信息   
>
> `man ls`
>
> 在 linux 下，隐藏文件是以 .开头 , 选项可以组合使用 比如 ls -al, 比如 `ls                                                 -al /root`

#### help 指令

- **基本语法**：`help 命令` 
- **功能描述**：获得 shell 内置命令的帮助信息

### 文档目录类

#### pwd 指令

- **功能描述**：显示当前工作目录的绝对路径

- **基本语法**：`pwd`

#### ls 指令

- 基本语法：`ls   [选项]  [目录或是文件]`

- **常用选项**：
  - `-a` ：显示当前目录所有的文件和目录，包括隐藏的。
  - `-l` ：以列表的方式显示信息
  - `-h` ：以人更容易理解的方式显示信息

#### cd 指令

- **功能描述**：切换到指定目录

- **基本语法**：`cd  [参数]`  

> `cd ~` 或者 `cd` ：回到自己的家目录, 比如 你是 root ，cd ~ 到 /root 
>
> `cd ..` 回到当前目录的上一级目录

#### mkdir 指令

- **功能描述**：创建目录

- **基本语法**：`mkdir [选项] 要创建的目录`

- **常用选项**：
  - `-p` ：创建多级目录

#### rmdir 指令

- **功能描述**：删除空目录

- **基本语法**：`rmdir [选项] 要删除的空目录`

> `rmdir` 删除的是空目录，如果目录下有内容时无法删除的。
>
> 提示：如果需要删除非空目录，需要使用 `rm -rf 要删除的目录`，比如： rm -rf /home/animal

#### touch 指令

- **功能描述**：创建空文件

- **基本语法**：`touch 文件名称`

#### cp 指令

- **功能描述**：拷贝文件或目录到指定目录

- **基本语法**：`cp [选项] source dest`

- **常用选项**：
  - `-r` ：递归复制整个文件夹

> 案例 1: 将 /home/hello.txt 拷贝到 /home/bbb 目录下
>
> `cp hello.txt /home/bbb`
>
> 案例 2: 递归复制整个文件夹，举例, 比如将 /home/bbb 整个目录， 拷贝到 /opt 
>
> `cp -r /home/bbb /opt`
>
> **强制覆盖不提示的方法：\cp , `\cp -r /home/bbb /opt`**

#### rm 指令

- **功能描述**：移除文件或目录

- **基本语法**：`rm [选项] 要删除的文件或目录`

- **常用选项**：
  - `-r` ：递归删除整个文件夹
  - `-f` ： 强制删除不提示

> 强制删除不提示的方法：带上 -f 参数即可

#### mv 指令

- **功能描述**：移动文件与目录或重命名

- **基本语法**：
  - 重命名：`mv oldNameFile newNameFile`
  - 移动文件：`mv /temp/movefile /targetFolder` 

#### cat 指令

- **功能描述**：查看文件内容

- **基本语法**：`cat [选项] 要查看的文件`

- **常用选项**：
  - `-n`：显示行号

> cat 只能浏览文件，而不能修改文件，为了浏览方便，一般会带上  管道命令 | more `cat -n /etc/profile | more` [进行交互]

#### more 指令

- **功能描述**：more 指令是一个基于 VI 编辑器的文本过滤器，它以全屏幕的方式按页显示文本文件的内容

- **基本语法**：`more 要查看的文件`

- **常用选项**：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/121.jpg)

#### less 指令

- **功能描述**：less 指令用来分屏查看文件内容，它的功能与 more 指令类似，但是比 more 指令更加强大，支持各种显示终端。less 指令在显示文件内容时，并不是一次将整个文件加载之后才显示，而是根据显示需要加载内容，对于显示大型文件具有较高的效率
- **基本语法**：`less 要查看的文件`

- **常用选项**：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/less.jpg)

#### echo 指令

- **功能描述**：输出内容到控制台
- **基本语法**：`echo [选项] [输出内容]`

#### head 指令

- **功能描述**：查看文件头 10 行内容
- **基本语法**：`head  文件`

- **常用选项**：
  - `-n num`：查看文件头 num 行内容

> 查看/etc/profile 的前面 5 行代码
>
> `head -n 5 /etc/profile`

#### tail 指令

- **功能描述**：用于输出文件中尾部的内容，默认情况下 tail 指令显示文件的前 10 行内容
- **基本语法**：
  - `tail 文件`：查看文件尾 10 行内容
  - `tail -n 5 文件`：查看文件尾 5 行内容，5 可以是任意行数
  - `tail -f  文件`：实时追踪该文档的所有更新

#### > 指令 和 >> 指令

- **功能描述**：`>` 输出重定向 ； `>>` 追加
- **基本语法**：
  - `ls -l >文件`：列表的内容写入文件中（覆盖写）
  - `ls -al >>文件`：列表的内容追加到文件的末尾
  - `cat 文件 1 > 文件 2`：将文件 1 的内容覆盖到文件 2
  - `echo "内容">> 文件`：追加

#### ln 指令

- **功能描述**：给原文件创建一个软链接
- **基本语法**：`ln -s [原文件或目录] [软链接名]`

> 案例 1: 在/home 目录下创建一个软连接 myroot，连接到 /root 目录
>
> `ln -s /root  /home/myroot`
>
> 案例 2: 删除软连接 myroot 
>
> `rm /home/myroot`
>
> **当我们使用 pwd 指令查看目录时，仍然看到的是软链接所在目录**

#### history 指令

- **功能描述**：查看已经执行过历史命令
- **基本语法**：`history`

> 案例 1: 显示所有的历史命令
>
> `history`
>
> 案例 2: 显示最近使用过的 10 个指令。
>
> `history 10`
>
> 案例 3：执行历史编号为 5 的指令
>
> `!5`

### 时间日期类

#### date 指令-显示当前日期

- **功能描述**：显示当前日期
- **基本语法**：
  - `date`：显示当前时间
  - `date +%Y`：显示当前年份
  - `date +%m`：显示当前月份
  - `date +%d`：示当前是哪一天
  - `date "+%Y-%m-%d %H:%M:%S"`：显示年月日时分秒

#### date 指令-设置日期

- **功能描述**：设置日期
- **基本语法**：`date -s 字符串时间`

> 案例 1: 设置系统当前时间 ， 比如设置成 2021-08-15 11:52:10
>
> `date -s “2021-08-15 11:52:10”`

#### cal 指令

- **功能描述**：查看日历
- **基本语法**：`cal [选项]`：不加选项，显示本月日历

> 案例 1: 显示当前日历
>
> `cal`
>
> 案例 2: 显示 2021 年日历 : 
>
> `cal 2021`
>
> 案例 3: 显示2021年5月份日历 : 
>
> `cal 5 2021`

### 搜索查找类

#### find 指令

- **功能描述**：将从指定目录向下递归地遍历其各个子目录，将满足条件的文件或者目录显示在终端
- **基本语法**：`find [搜索范围] [选项]`

- **常用选项**：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815115849.png)

> 案例 1: 按文件名：根据名称查找**/home** 目录下的 **hello.txt** 文件
>
> `find /home -name hello.txt`
>
> 案例 2：按拥有者：查找**/opt** 目录下，用户名称为 **nobody** 的文件
>
> `find /opt -user nobody`
>
> 案例 3：查找整个 linux 系统下**大于 200M** 的文件（+n  大于，-n 小于，n 等于, 单位有 k,M,G） 
>
> `find / -size +200M`

#### locate 指令

- **功能描述**：locate 指令可以快速定位文件路径。locate 指令利用事先建立的系统中所有文件名称及路径的 locate 数据库实现快速定位给定的文件。Locate 指令无需遍历整个文件系统，查询速度较快。为了保证查询结果的准确度，管理员必须定期更新 locate 时刻
- **基本语法**：`locate 搜索文件`

> **由于 locate 指令基于数据库进行查询，所以第一次运行前，必须使用 `updatedb` 指令创建 locate 数据库。**

#### which 指令

- **功能描述**：可以查看某个指令在哪个目录下
- **基本语法**：`which 指令`：

> ls 指令在哪个目录
>
> `which ls`

#### grep 指令和 管道符号 |

- **功能描述**：grep 过滤查找 ， 管道符，“|”，表示将前一个命令的处理结果输出传递给后面的命令处理
- **基本语法**：`grep [选项] 查找内容 源文件`

- **常用选项**：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815120451.png)

> 案例 1: 请在 **hello.txt** 文件中，查找   **"yes"**所在行，并且显示行号
>
> - 写法 1: `cat /home/hello.txt | grep -n "yes"`
>
> - 写法 2: `grep -n "yes" /home/hello.txt`

### 压缩和解压类

#### gzip/gunzip 指令

- **功能描述**：`gzip` 用于压缩文件， `gunzip` 用于解压的
- **基本语法**：
  - `gzip 文件`：压缩文件，只能将文件压缩为`*.gz` 文件
  - `gunzip 文件.gz`：解压缩文件命令

> 案例 1: `gzip`压缩， 将 **/home** 下的 **hello.txt** 文件进行压缩
>
> `gzip  /home/hello.txt`
>
> 案例 2: `gunzip`解压缩， 将 **/home** 下的 **hello.txt.gz** 文件进行解压缩
>
> `gunzip /home/hello.txt.gz`

### zip/unzip 指令

- **功能描述**：`zip` 用于压缩文件， `unzip` 用于解压的，这个在项目打包发布中很有用的
- **基本语法**：
  - `zip [选项] XXX.zip 将要压缩的内容`：压缩文件和目录的命令
  - `unzip [选项] XXX.zip`：解压缩文件

- **zip常用选项**：
  - `-r`：递归压缩，即压缩目录

- **unzip常用选项**：
  - `-d<目录>` ：指定解压后文件的存放目录

> 案例1:将 **/home** 下的 所有文件/文件夹进行压缩成 **myhome.zip**
>
> `zip -r myhome.zip /home/`  [将 home 目录及其包含的文件和子文件夹都压缩] 
>
> 案例2: 将 **myhome.zip** 解压到 **/opt/tmp** 目录下
>
> `mkdir /opt/tmp`
>
> `unzip -d /opt/tmp /home/myhome.zip`

### tar 指令

- **功能描述**：tar 指令是打包指令，最后打包后的文件是 `.tar.gz` 的文件
- **基本语法**：`tar [选项]  XXX.tar.gz 打包的内容`：打包目录，压缩后的文件格式`.tar.gz`

- **常用选项**：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815130556.png)

> 案例 1: 压缩多个文件，将 **/home/pig.txt** 和 **/home/cat.txt** 压缩成**pc.tar.gz** 
>
> `tar -zcvf pc.tar.gz /home/pig.txt /home/cat.txt`
>
> 案例 2:将**/home** 的文件夹 压缩成 **myhome.tar.gz** 
>
> `tar -zcvf myhome.tar.gz /home/`
>
> 案例 3:    将 pc.tar.gz  解压到当前目录
>
> `tar -zxvf pc.tar.gz`
>
> 案例4: 将**myhome.tar.gz** 解压到 **/opt/tmp2** 目录下 
>
> - `mkdir /opt/tmp2` 
>
> - `tar -zxvf /home/myhome.tar.gz -C /opt/tmp2`

## 组管理和权限管理

### 修改文件所有者

- **基本语法**：
  - `chown newowner 文件/目录`：改变所有者
  - `chown newowner:newgroup 文件/目录`：改变所有者和所在组
- **常用选项**：
  - `-R` ：如果是目录 则使其下所有子文件或目录递归生效

> 请将 /home/abc.txt 文件的所有者修改成 tom 
>
> `chown tom /home/abc.txt`
>
> 请将 /home/test 目录下所有的文件和目录的所有者都修改成 tom 
>
> `chown -R tom /home/test`

### 修改文件/目录所在的组

- **基本语法**：`chgrp newgroup 文件/目录`
- **常用选项**：
  - `-R` ：如果是目录 则使其下所有子文件或目录递归生效

### 改变用户所在组

- **基本语法**：

  - `usermod –g 新组名 用户名`
  - `usermod –d 目录名 用户名`：改变该用户登陆的初始目录

  > 用户需要有进入到新目录的权限

### 修改权限

#### 第一种方式：\+ 、-、= 变更权限

- **基本语法**：u:所有者；g:所有组；o:其他人；a:所有人(u、g、o 的总和)

  - `chmod u=rwx,g=rx,o=x 文件/目录名`

  - `chmod o+w 文件/目录名`

  - `chmod a-x 文件/目录名`

> 给 abc 文件 的所有者读写执行的权限，给所在组读执行权限，给其它组读执行权限。
>
> `chmod u=rwx,g=rx,o=rx abc`
>
> 给 abc 文件的所有者除去执行的权限，增加组写的权限
>
> `chmod u-x,g+w abc`
>
> 给 abc 文件的所有用户添加读的权限
>
> `chmod a+r abc`

#### 第二种方式：通过数字变更权限

- **基本语法**：r=4 w=2 x=1     rwx=4+2+1=7
  -  `chmod 751 文件/目录名`：相当于`chmod u=rwx,g=rx,o=x 文件/目录名`

> 将 /home/abc.txt 文件的权限修改成 `rwxr-xr-x`， 使用给数字的方式实现：
>
> `chmod 755 /home/abc.txt`

## 定时任务调度

任务调度：是指系统在某个时间执行的特定的命令或程序。    
任务调度分类：1.系统工作：有些重要的工作必须周而复始地执行。如病毒扫描等个别用户工作：个别用户可能希望执行某些程序，比如对 mysql 数据库的备份。

示意图：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815141316.png)

### crond 任务调度

- **功能描述**：crontab 进行定时任务的设置
- **基本语法**：`crontab [选项]`

- **常用选项**：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815141149.png)

> 设置任务调度文件：/etc/crontab
>
> 设置个人任务调度。执行 crontab –e 命令。接着输入任务到调度文件
>
> 如：`*/1 * * * * ls –l  /etc/ > /tmp/to.txt`
>
> 意思说每小时的每分钟执行 `ls –l /etc/ > /tmp/to.txt` 命令

- **参数细节说明**：

5个占位符的说明

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815141502.png)

- **特殊符号的说明**

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815141536.png)

- **特殊时间执行案例**：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815141712.png)

- **应用案例：**

> - 案例 1：每隔 1 分钟，就将当前的日期信息，追加到 /tmp/mydate 文件中
>
> `*/1 * * * * date >> /tmp/mydate`
>
> - 案例 2：每隔 1 分钟， 将当前日期和日历都追加到 /home/mycal 文件中步骤:
>
> 1. `vim /home/my.sh`  写入内容 `date >> /home/mycal` 和 `cal >> /home/mycal`
>
> 2. 给 my.sh 增加执行权限，`chmod u+x /home/my.sh`
>
> 3. `crontab -e`    增加 `*/1 * * * * /home/my.sh`
>
> - 案例 3:   每天凌晨 2:00 将 mysql 数据库 testdb ，备份到文件中。提示: 指令为
>
> `mysqldump -u root -p 密码 数据库 > /home/db.bak`
>
> 1. `crontab -e`
>
> 2. `0 2 * * * mysqldump -u root -proot testdb > /home/db.bak`

- **crond 相关指令**
  - `conrtab –r`：终止任务调度。
  - `crontab –l`：列出当前有那些任务调度
  - `service crond restart`   [重启任务调度]

### at 定时任务

- **基本介绍：**

1. at 命令是一次性定时计划任务，at 的守护进程 atd 会以后台模式运行，检查作业队列来运行。

2. 默认情况下，atd 守护进程每 60 秒检查作业队列，有作业时，会检查作业运行时间，如果时间与当前时间匹配，则运行此作业。

3. at 命令是一次性定时计划任务，执行完一个任务后不再执行此任务了

4. 在使用 at 命令的时候，一定要保证 atd 进程的启动 , 可以使用相关指令来查看`ps -ef | grep atd` //可以检测 atd 是否在运行

5. 画一个示意图

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815142510.png)

- **at命令格式**
  - `at [选项] [时间]`
  - `Ctrl + D`  结束 at 命令的输入， 输出两次

- **at命令选项**

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815142740.png)

- **at 时间定义**

at 指定时间的方法：

1. 接受在当天的 hh:mm（小时:分钟）式的时间指定。假如该时间已过去，那么就放在第二天执行。 例如：04:00

2. 使用 midnight（深夜），noon（中午），teatime（饮茶时间，一般是下午 4 点）等比较模糊的词语来指定时间。

3. 采用 12 小时计时制，即在时间后面加上 AM（上午）或 PM（下午）来说明是上午还是下午。 例如：12pm

4. 指定命令执行的具体日期，指定格式为 month day（月 日）或 mm/dd/yy（月/日/年）或 dd.mm.yy（日.月.年），指定的日期必须跟在指定时间的后面。 例如：04:00 2021-03-1

5. 使用相对计时法。指定格式为：now + count time-units ，now 就是当前时间，time-units 是时间单位，这里能够是 minutes（分钟）、hours（小时）、days（天）、weeks（星期）。count 是时间的数量，几天，几小时。 例如：now + 5 minutes

6. 直接使用 today（今天）、tomorrow（明天）来指定完成命令的时间。

- **应用实例**

> 案例 1：2 天后的下午 5 点执行 /bin/ls /home

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815144150.png)

> 案例 2：`atq` 命令来查看系统中没有执行的工作任务

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815144222.png)

> 案例 3：明天 17 点钟，输出时间到指定文件内 比如 /root/date100.log

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815144454.png)

> 案例 4：2 分钟后，输出时间到指定文件内 比如 /root/date200.log

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210815144654.png)

> 案例 5：删除已经设置的任务 , atrm 编号
>
> atrm 2 //表示将 job 队列，编号为 2 的 job 删除

