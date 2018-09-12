##### **Linux系统操作命令整理**

Linux整个系统共分为四层：

- 硬件
- 内核
- Shell
- 应用程序


内核加Shell就是操作系统，即操作系统就是由内核和Shell构成。

内核的作用是啥呢？内核是负责对我们计算机硬件进行抽象，然后通过系统调用和库文件为上层应用提供服务。

Shell就是我们人机交互的窗口。就像我们的微软系统有个桌面，Shell就相当于这个桌面，只不过Shell是命令行模式而不是图形模式。

Linux的命令格式是：命令字 -选项（参数）操作对象。

例如：mkdir -P /root/mulu

**注意：Linux命令严格区分大小写**

##### **文件目录命令**

- touch：创建文件
- mkdir：创建目录
- cd：切换目录
- mv：移动目录或者文件
- cp：复制文件或者目录
- rm：删除文件或者目录

##### **查看类命令：**

- cat：查看某个文件内容。cat /etc/passwd
- more：more /etc/passwd
- less：less /etc/passwd
- head：head -n /etc/passwd
- tail：tail -n /etc/passwd

##### **查找类命令：**

- which：查找某个可执行文件的位置或者目录。which cd
- find：根据特定条件查找。find / -name cd
- whereis：查找二进制文件，说明文档的目录。whereis cd
- locate：也是查找某个文件的目录，不常用。

##### **帮助类命令：**

- help：查看内置命令的是用。help cd
- man：默认情况下是查命令。man --help
- info：info提供的信息通常要比man所显示的信息更加完整些。info mkdir
- --help：一个简略的命令帮助。passwd --help

##### **系统信息类命令：**

- clear：清屏
- hostname：查看计算机名
- uname：查看内核版本信息
- history：查看命令历史记录
- alias：查看命令别名
- halt：关机
- reboot：重启
- free：查看内存使用情况
- uptime：查看系统运行时间，用户数量，负载等情况

##### **用户组类命令：**

- useradd：创建用户。
- usermod：修改用户信息。
- userdell：删除用
- passwd：给用户设置密码。passwd 123456
- finger：查看单个用户信息。
- id：查看用户的id号。
- su：切换用户。su -lele
- who：查看当前登陆的所有用户
- groupadd：添加用户组。
- groupdel：删除用户组
- groupmod：修改用户组信息

##### **磁盘类命令：**

- mount：挂载命令。
- fdisk：显示系统所有的分区。fdisk －l
- fdisk：显示系统所有分区，显示的是扇区数而不是柱面数。fdisk －u
- mkfs：对某个磁盘分区建立文件系统。
- parted：大于2T的磁盘分区。

##### **网络类命令：**

- ifconfig：查看ip地址命令。
- ping：检查网络联通命令。
- traceroute：路由跟踪命令。
- nslookup：域名解析命令。
- route：设置路由命令。

##### **进程类命令：**

- top：动态查看进程。
- ps：静态查看进程。
- Kill，pkill，killall：杀死进程。

##### **安装类命令：**

- rpm安装命令：
  - rpm -ihv pakcage.rpm：其中i的意思是install安装；v是显示到前台；h是显示进度信息。
  - rpm -qa：q表示查询query，a表示所有
  - rpm -e：e表示卸载

- tar源码安装方法：
  - tar xzf package.tar：解压，x的意思是解开压缩文件的参数，z表示打包后用gzip压缩的，f是使用文档名，j表示用bzip2压缩的。
- yum安装方法：
  - yum install package：安装package
  - yum list：显示可用软件包
  - yum list installed：显示安装了的软件包
  - yum info：获取软件包信息

##### **时间类命令：**

- date：跟时间相关的。