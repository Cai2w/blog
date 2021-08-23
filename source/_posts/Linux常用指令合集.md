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

- **基本语法**：`ls   [选项]  [目录或是文件]`

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

## 磁盘分区、挂载

### Linux分区

#### 原理介绍

- Linux 来说无论有几个分区，分给哪一目录使用，它归根结底就只有一个根目录，一个独立且唯一的文件结构 , Linux中每个分区都是用来组成整个文件系统的一部分
- Linux 采用了一种叫“载入”的处理方法，它的整个文件系统中包含了一整套的文件和目录，且将一个分区和一个目录联系起来。这时要载入的一个分区将使它的存储空间在一个目录下获得
- 示意图

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817175124.png)

#### 硬盘说明

- Linux 硬盘分 IDE 硬盘和 SCSI 硬盘，目前基本上是 SCSI 硬盘
- 对于 IDE 硬盘，驱动器标识符为“hdx\~”,其中“hd”表明分区所在设备的类型，这里是指 IDE 硬盘了。“x”为盘号（a 为基本盘，b 为基本从属盘，c 为辅助主盘，d 为辅助从属盘）,“\~”代表分区，前四个分区用数字 1 到 4 表示，它们是主分区或扩展分区，从 5 开始就是逻辑分区。例，hda3 表示为第一个 IDE 硬盘上的第三个主分区或扩展分区,hdb2 表示为第二个 IDE 硬盘上的第二个主分区或扩展分区
- 对于 SCSI 硬盘则标识为“sdx~”，SCSI 硬盘是用“sd”来表示分区所在设备的类型的，其余则和 IDE 硬盘的表示方法一样

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817175928.png)

#### 查看所有设备挂在情况

- 命令：
  - `lsblk`
  - `lsblk -f`

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817175928.png)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817180149.png)

> 其中`sdax`为分区情况，`MOUNTPOINT`为挂载目录

### 挂载案例

#### 说明

下面我们以**增加一块硬盘**为例来熟悉下磁盘的相关指令和深入理解磁盘分区、挂载、卸载的概念

#### 如何增加一块硬盘

1. 虚拟机添加硬盘
2. 分区
3. 格式化
4. 挂载
5. 设置可以自动挂载

#### 虚拟机增加硬盘步骤1

> 在【虚拟机】菜单中，选择【设置】，然后设备列表里添加硬盘，然后一路【下一步】，中间只有选择磁盘大小的地方需要修改，至到完成。然后重启系统（才能识别）

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817180815.png)

#### 虚拟机增加硬盘步骤2

- 分区命令：`fdisk /dev/sdb`

开始对`/sdb`分区

- `m`：显示命令列表
- `p`：显示磁盘分区 同 `fdisk –l` 
- `n`：新增分区
- `d`：删除分区
- `w`：写入并退出

> 说明：开始分区后输入 `n`，新增分区，然后选择 `p` ，分区类型为主分区。两次回车默认剩余全部空间。最后输入 `w` 写入分区并退出，若不保存退出输入 `q`

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817181503.png)

#### 虚拟机增加硬盘步骤3

- 格式化磁盘：`mkfs -t ext4 /dev/sdb1`

> 其中`ext4`是分区类型

#### 虚拟机增加硬盘步骤4

- 挂载：将一个分区与一个目录联系起来
- 挂载命令：`mount 设备名称 挂载目录`

> 例如：`mount /dev/sdb1 /newdisk`

- 卸载命令：
  - `umount 设备名称`
  - `umount 挂载目录`

> 例如：`umount /dev/sdb1` 或者 `umount /newdisk`

> **注意：用命令行挂载,重启后会失效**

#### 虚拟机增加硬盘步骤5

永久挂载：通过修改`/etc/fstab`实现挂载

- 命令：`vim /etc/fstab`

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817183202.png)

添加完成之后，执行`mount –a`即刻生效

> 其中，后面的第一个数字表示指定文件系统是否需要使用dump进行备份 (0 为不备份，1 为要备份, 一般根分区要备份)；第二个数字表示指定文件系统将按照何种顺序来自动检查错误和损坏 (0 -不自检，1 或者 2 -要自检;如果是根分区要设为1，其他分区只能是2)

### 磁盘情况查询

#### 查询系统整体磁盘使用情况

- 基本语法：`df -h`
- 应用实例

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817183914.png)

#### 查询指定目录的磁盘占用情况

- 基本语法：`du -h`：查询指定目录的磁盘占用情况，默认为当前目录

- 常用选项：
  - `-s`：指定目录占用大小汇总
  - `-h`：带计量单位
  - `-a`：含文件
  - `--max-depth=1`：子目录深度
  - `-c`：列出明细的同时，增加汇总值

- 应用实例：查询 /opt 目录的磁盘占用情况，深度为

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817184603.png)

### 磁盘情况-工作实用指令

- 统计/opt 文件夹下文件的个数

> `ls -l /opt | grep "^-" | wc -l`

- `统计/opt 文件夹下目录的个数`

> `ls -l /opt | grep "^d" | wc -l`

- 统计/opt 文件夹下文件的个数，包括子文件夹里的

> `ls -lR /opt | grep "^-" | wc -l`

- 统计/opt 文件夹下目录的个数，包括子文件夹里的

> `ls -lR /opt | grep "^d" | wc -l`

- 以树状显示目录结构 tree 目录 ， 注意，如果没有 tree ,则使用 **yum install tree** 安装

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210817185319.png)

## 网络配置

### 原理图

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820115925.png)

### 查看网络IP和网关

#### 查看虚拟网络编辑器和修改IP 地址

VMware：编辑—>虚拟网络编辑器

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820115355.png)

#### 查看网关

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820115453.png)

### 查看 windows 环境的中 VMnet8 网络配置 (ipconfig 指令)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820115631.png)

### 查看 linux 的网络配置 ifconfig

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820115853.png)

### ping 测试主机之间网络连通性

#### 基本语法

ping 目的主机 （功能描述：测试当前服务器是否可以连接目的主机）

#### 应用实例

测试当前服务器是否可以连接百度

ping [www.baidu.com](http://www.baidu.com/)

### linux 网络环境配置

#### 第一种方法（自动获取）

说明：登陆后，通过界面的来设置自动获取ip，特点：linux 启动后会自动获取 IP,缺点是每次自动获取的 ip 地址可能不一样

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820141935.png)

#### 第二种方法（指定ip）

* 说明：直接修改配置文件来指定 IP,并可以连接到外网(程序员推荐)

```bash
# 编辑网络配置文件
vim /etc/sysconfig/network-scripts/ifcfg-ens33
要求：将 ip 地址配置的静态的，比如: ip 地址为 192.168.200.130

# ifcfg-ens33 文件说明
DEVICE=eth0	#接口名（设备,网卡） 
HWADDR=00:0C:2x:6x:0x:xx	#MAC 地址
TYPE=Ethernet	#网络类型（通常是 Ethemet） 
UUID=926a57ba-92c6-4231-bacb-f27e5e6a9f44	#随机 id
#系统启动的时候网络接口是否有效（yes/no） 
ONBOOT=yes
```

- 修改ip地址为静态

```bash
# IP 的配置方法[none|static|bootp|dhcp]（引导时不使用协议|静态分配 IP|BOOTP 协议|DHCP 协议）
BOOTPROTO=static 
#IP 地址
IPADDR=192.168.200.130
#网关
GATEWAY=192.168.200.2
#域名解析器
DNS1=192.168.200.2

# 重启网络服务或者重启系统生效
service	network restart	# 重启网络
reboot # 重启主机
```

### 设置主机名和hosts 映射

#### 设置主机名

> 为了方便记忆，可以给 **linux** **系统设置主机名**, 也可以根据需要修改主机名

- 指令：`hostname`：查看主机名

> 修改文件在 **/etc/hostname** 指定
>
> 修改后，**重启**生效

#### 设置hosts 映射

> - **windows**：在 **C:\Windows\System32\drivers\etc\hosts** 文件指定即可
>
> 案例: 192.168.200.130 caicai
>
> linux：在 **/etc/hosts** 文件 指定
>
> 案例: 192.168.200.1 myWindows

### 主机名解析过程分析(Hosts、DNS)

#### Hosts是什么

一个文本文件，用来**记录** **IP** 和 **Hostname(主机名)**的映射关系

#### DNS

**DNS**，就是 **Domain Name System** 的缩写，翻译过来就是域名系统是互联网上作为域名和 IP 地址相互映射的一个**分布式数据**库

#### 应用实例：用户在浏览器输入了www.baidu.com

- 浏览器先检查浏览器缓存中有没有该域名解析 IP 地址，有就先调用这个 IP 完成解析；如果没有，就检查 DNS 解析器缓存，如果有直接返回 IP 完成解析。这两个缓存，可以理解为**本地解析器缓存**
- 一般来说，当电脑第一次成功访问某一网站后，在一定时间内，浏览器或操作系统会缓存他的 IP 地址（DNS 解析记录）.如 在 cmd 窗口中输入

```bash
ipconfig /displaydns	#DNS 域名解析缓存
ipconfig /flushdns	#手动清理 dns 缓存
```

- 如果本地解析器缓存没有找到对应映射，检查系统中`hosts`文件中有没有配置对应的域名 IP 映射，如果有，则完成解析并返回

- 如果本地`DNS`解析器缓存和`hosts` 文件中均没有找到对应的 IP，则到域名服务DNS进行解析域

- 示意图



![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820151852.png)

## 进程管理

### 基本介绍

1. 在 LINUX 中，每个执行的程序都称为一个进程。每一个进程都分配一个 ID 号(pid,进程号)。
2. 每个进程都可能以两种方式存在的。前台与后台，所谓前台进程就是用户目前的屏幕上可以进行操作的。后台进程则是实际在操作，但由于屏幕上无法看到的进程，通常使用后台方式执行
3. 一般系统的服务都是以后台进程的方式存在，而且都会常驻在系统中。直到关机才才结束
4. 示意图：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820154056.png)

### 显示系统执行的进程

#### 基本介绍

`ps` 命令是用来查看目前系统中，有哪些正在执行，以及它们执行的状况。可以不加任何参数

- 常用参数：
  - `-a`：显示当前终端的所有进程信息
  - `-u`：以用户的格式显示进程信息
  - `-x`：显示后台进程运行的参数

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820155939.png)

#### ps详解

* 指令：`ps -aux|grep xxx`：
* 指令说明：
  * System V 展示风格
  * USER：用户名称
  * PID：进程号
  * %CPU：进程占用 CPU 的百分比
  * %MEM：进程占用物理内存的百分比
  * VSZ：进程占用的虚拟内存大小（单位：KB）
  * RSS：进程占用的物理内存大小（单位：KB）
  * TT：终端名称,缩写 .
  * STAT：进程状态，其中 S-睡眠，s-表示该进程是会话的先导进程，N-表示进程拥有比普通优先级更低的优先级，R-正在运行，D-短期等待，Z-僵死进程，T-被跟踪或者被停止等等
  * STARTED：进程的启动时间
  * TIME：CPU 时间，即进程使用 CPU 的总时间
  * COMMAND：启动进程所用的命令和参数，如果过长会被截断显示

#### 应用实例

要求：以全格式显示当前所有的进程，查看进程的父进程。 查看 sshd 的父进程信息

`ps -ef` 是以全格式显示当前所有的进程

`-e` 显示所有进程。`-f` 全格式

```bash
ps -ef|grep sshd
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820161044.png)

- UID：用户ID
- PID：进程ID
- PPID：父进程ID
- C：CPU 用于计算执行优先级的因子。数值越大，表明进程是 CPU 密集型运算，执行优先级会降低；数值越小，表明进程是 I/O 密集型运算，执行优先级会提高
- STIME：进程启动的时间
- TTY：完整的终端名称
- TIME：CPU 时间
- CMD：启动进程所用的命令和参数

### 终止进程kill和killall

#### 介绍

若是某个进程执行一半需要停止时，或是已消了很大的系统资源时，此时可以考虑停止该进程。使用 kill 命令来完成此项任务

#### 基本语法

- `kill [选项] 进程号`：通过进程号杀死/终止进程
- `killall 进程名称`：通过进程名称杀死进程，也支持通配符，这在系统因负载过大而变得很慢时很有用

#### 常用选项

`-9`：表示强迫进程立即停止

#### 最佳实践

> 案例 1：踢掉某个非法登录用户

```bash
# kill 进程号 
kill 11421
```

> 案例 2: 终止远程登录服务 sshd, 在适当时候再次重启 sshd 服务

```bash
# 终止远程登录服务
kill sshd对应的进程号

# 再次重启 sshd 服务
/bin/systemctl start sshd.service
```

> 案例 3: 终止多个 gedit 

```bash
killall	gedit
```

> 案例 4：强制杀掉一个终端

```bash
kill -9 bash对应的进程号
```

### 查看进程树 pstree

#### 基本语法

`pstree [选项]`：可以更加直观的来看进程信息

#### 常用选项

- `-p`：显示进程的PID
- `-u`：显示进程的所属用户

#### 应用实例

> 案例 1：请你树状的形式显示进程的 pid 

```bash
pstree -p
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820165429.png)

> 案例 2：请你树状的形式进程的用户

```bash
pstree -u
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820165403.png)

### 服务（service）管理

#### 介绍

服务(service) 本质就是进程，但是是运行在后台的，通常都会监听某个端口，等待其它程序的请求，比如(mysqld , sshd，防火墙等)，因此我们又称为守护进程，是 Linux 中非常重要的知识点

#### service 管理指令

* `service 服务名 [start | stop | restart | reload | status]`
* 在 CentOS7.0 后 (我的是7.6)，很多服务不再使用 service ,而是 systemctl (后面专门讲)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820165803.png)

* service 指令管理的服务在 `/etc/init.d` 查看

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820165938.png)

#### service管理指令案例

> 使用 service 指令，查看，关闭，启动 network [注意：在虚拟系统演示，因为网络连接会关闭]

```bash
# 查看
service network status 
# 关闭
service network stop 
# 启动
service network start
```

#### 查看服务名

* 方式一：使用 setup -> 系统服务 就可以看到全部

```bash
setup
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820170311.png)

**\*表示开机自启动**

* 方式二：`/etc/init.d` 看到 service 指令管理的服务

```bash
ls -l /etc/init.d
```

#### 服务的运行级别(runlevel)

* Linux 系统有 7 种运行级别(runlevel)：常用的是**级别** **3** **和** **5**
  * 运行级别 0：系统停机状态，系统默认运行级别不能设为 0，否则不能正常启动运行级别 1：单用户工作状态，root 权限，用于系统维护，禁止远程登陆
  * 运行级别 2：多用户状态(没有 NFS)，不支持网络
  * 运行级别 3：完全的多用户状态(有 NFS)，无界面，登陆后进入控制台命令行模式
  * 运行级别 4：系统未使用，保留
  * 运行级别 5：X11 控制台，登陆后进入图形 GUI 模式
  * 运行级别 6：系统正常关闭并重启，默认运行级别不能设为 6，否则不能正常启动

* 开机流程说明：

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820170658.png)

#### CentOS7 后运行级别说明

在 `/etc/inittab`中

```bash
cat /etc/inittab
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820171023.png)

#### chkconfig 指令

通过 chkconfig 命令可以给服务的各个运行级别设置自 启动/关闭

chkconfig 指令管理的服务在 /etc/init.d 查看   

> 注意: Centos7.0 后，很多服务使用 systemctl 管理

* 基本语法：
  * `chkconfig [服务名] --list [| grep xxx]`：查看服务
  * `chkconfig --level 5 服务名  on/off`：设置服务在5运行级别的自启动

* 案例演示：对 network  服务    进行各种操作, 把 network 在 3 运行级别,关闭自启动

```bash
chkconfig --level 3 network off 
chkconfig --level 3 network on
```

> `chkconfig` 重新设置服务后自启动或关闭，需要重启机器 `reboot` 生效

#### systemctl 管理指令

- 基本语法：`systemctl [start | stop | restart | status] 服务名`

> systemctl 指令管理的服务在 /usr/lib/systemd/system 查看

#### systemctl 设置服务的自启动状态

```bash
# 查看服务开机启动状态, grep 可以进行过滤
systemctl list-unit-files [ | grep 服务名]
# 设置服务开机启动
systemctl enable 服务名
# 关闭服务开机启动
systemctl disable 服务名
# 查询某个服务是否是自启动的
systemctl is-enabled 服务名
```

#### 应用实例

> 查看当前防火墙的状况，关闭防火墙和重启防火墙。=> firewalld.service

```bash
# 查看
systemctl status firewalld

# 关闭
systemctl stop firewalld 

# 重启
systemctl start firewalld
```

#### 细节讨论

关闭或者启用防火墙后，立即生效。[telnet 测试 某个端口即可]

```bash
# 在windows上，测试111端口
telnet 192.168.200.130 111
```

这种方式只是临时生效，当重启系统后，还是回归以前对服务的设置。

如果希望设置某个服务自启动或关闭永久生效，要使用 systemctl [enable|disable] 服务名

#### 打开或者关闭指定端口

在真正的生产环境，往往需要将防火墙打开，但问题来了，如果我们把防火墙打开，那么外部请求数据包就不能跟服务器监听端口通讯。这时，需要打开指定的端口。比如 80、22、8080 等

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820174627.png)

#### firewall 指令

* 打开端口：`firewall-cmd --permanent --add-port=端口号/协议`

* 关闭端口：`firewall-cmd --permanent --remove-port=端口号/协议`

* 重新载入才能生效：`firewall-cmd --reload`
* 查询端口是否开放：`firewall-cmd --query-port=端口/协议`

#### 应用案例

* 启动防火墙，测试111端口是否能telnet：不可以
* 开放111端口

```bash
firewall-cmd --permanent --add-port=111/tcp 
firewall-cmd --reload
```

* 再次关闭111端口

```bash
firewall-cmd --permanent --remove-port=111/tcp 
firewall-cmd --reload
```

### 动态监控进程

#### 介绍

top 与 ps 命令很相似。它们都用来显示正在执行的进程。Top 与 ps 最大的不同之处，在于 top 在执行一段时间可以更新正在运行的的进程

#### 基本语法

```bash
top [选项]
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820175417.png)

#### 选项说明

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820175100.png)

#### 交互操作说明

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820175643.png)

#### 应用实例

* 案例 1.监视特定用户, 比如我们监控 tom 用户

> top：输入top命令，按回车键，查看执行的进程
>
> u：然后输入“u”回车，再输入用户名，即可

* 案例 2：终止指定的进程, 比如我们要结束 tom 登录

> top：输入此命令，按回车键，查看执行的进程
>
> k：然后输入“k”回车，再输入要结束的进程 ID 号

* 案例 3:指定系统状态更新的时间(每隔 10 秒自动更新), 默认是 3 秒

> top -d 10

### 监控网络状态

#### 查看系统网络情况netstat

* 基本语法：`netstat [选项]`

* 选项说明：
  * `-an`：按一定顺序排列输出
  * `-p`：显示哪个进程在调用

* 应用案例

请查看服务名为 sshd 的服务的信息

```bash
netstat -anp | grep sshd
```

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820180252.png)

![](https://cdn.jsdelivr.net/gh/Cai2w/cdn/img/20210820180328.png)

#### 检测主机连接命令 ping

是一种网络检测工具，它主要是用检测远程主机是否正常，或是两部主机间的网线或网卡故障。如: ping 对方 ip 地址

## RPM 与 YUM

### rpm包的管理

#### 介绍

> rpm 用于互联网下载包的打包及安装工具，它包含在某些 Linux 分发版中。它生成具有.RPM 扩展名的文件。RPM 是 RedHat Package Manager（RedHat 软件包管理工具）的缩写，类似 windows 的setup.exe，这一文件格式名称虽然打上了 RedHat 的标志，但理念是通用的。
>
> Linux 的分发版本都有采用（suse,redhat, centos 等等），可以算是公认的行业标准了。

#### rpm 包的简单查询指令

- 查询已安装的rpm列表：`rpm -qa | grep xx`

> 举例：查看当前系统是否安装了firefox
>
> `rpm -qa | grep firefox`

#### rpm包名基本格式

一个 rpm 包名：`firefox-60.2.2-1.el7.centos.x86_64`

版本号：60.2.2-1

适用操作系统: el7.centos.x86_64

x86_64表示 centos7.x 的 64 位系统，如果是 i686、i386 表示 32 位系统，noarch 表示通用

#### rpm包的其他查询指令

- 查询所安装的所有 rpm 软件包：`rpm -qa`

> `rpm -qa | more`
>
> `rpm -qa | grep X [rpm -qa | grep firefox ]`

- 查询软件包是否安装：`rpm -q 软件包名`

- 查询软件包信息：`rpm -qi 软件包名`

> 案例: `rpm -qi firefox`

- 查询软件包中的文件：`rpm -ql`

> `rpm -ql firefox`

- 查询文件所属的软件包：`rpm -qf 文件全路径名`

> `rpm -qf /etc/passwd`
>
> `rpm -qf /root/install.log`

#### 卸载rpm包































## Shell编程



## 日志管理



## 备份与恢复