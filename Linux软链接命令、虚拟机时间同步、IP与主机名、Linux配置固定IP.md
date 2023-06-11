## 1 `ln`命令创建软链接

在系统中创建软连接，可以将文件、文件夹链接到其他位置。

类似Windows系统中的《快捷方式》。

命令：`ln -s 参数1 参数2`

- -s选项，创建软链接
- 参数1：被链接的文件或文件夹
- 参数2：要链接去的目的地

实例：

- `ln -s /etc/yum.conf~/yum.conf`

![image-20230330164731445](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303301649584.png)

- `ln -s /etc/yum~/yum`

![image-20230330165629619](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303301656750.png)

## 日期和时区

### date命令

通过date命令可以在命令行中查看系统的时间

命令：`date [-d] [+格式化字符串]`

- -d，按照给定的字符串显示日期，一般用于日期计算
- 格式化字符串：通过特定的字符串标记，来控制显示的日期格式
  - %Y 年
  - %y 年份后两位数字（00-99）
  - %M 月份（01-12）
  - %d 日 （01-31）
  - %H 小时（00-23）
  - %M 分钟（00-59）
  - %S 秒 （00-60）
  - %s 自 1970-01-01 00:00:00 UTC 到现在的秒数 （也就是时间戳）

- 使用date命令本体，无选项，直接查看时间（如果时间与主机不同步，可能是因为时区不同原因。修改时区：`timedatectl set-timezone Asia/Shanghai`）

![image-20230402112206781](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022218653.png)

可以看到这个格式非常的不习惯，我们可以通过格式化字符串自定义显示格式。

- 按照2023-04-02的格式显示日期

![image-20230402112755886](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021127920.png)

- 按照2023-04-02 10:00:00的格式显示日期

![image-20230402113454761](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021134792.png)

如上，由于中间带有空格，所以**使用双引号包围格式化字符串，作为整体。**

- -d选项，可以按照给定的字符串显示日期，一般用于日期计算
  - `date -d "+1 day" +%Y%m%d`	#显示后一天的日期
  - `date -d "-1 day" +%Y%m%d`     #显示前一天的日期
  - `date -d "-1 month" +%Y%m%d`    #显示上一月的日期
  - `date -d "+1 month" +%Y%m%d`   #显示下一月的日期
  - `date -d "-1 year" +%Y%m%d`    #显示前一年的日期
  - `date -d "+1 year" +%Y%m%d `  #显示下一年的日期

- 其中支持的时间标记为：
  - year 年
  - Month 月
  - day 天
  - hour 小时
  - Minute 分钟
  - second 秒

- -d选项可以和格式化字符串配合一起使用哦

## 2 修改Linux时区

:另一种修改方式

系统默认时区非中国的东八区。

<font color="red">使用root权限，执行如下命令</font>，修改时区为东八区时区。

`rm -f /etc/localtime`

`sudo ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`

将系统自带的localtime文件删除，并将/usr/share/zoneinfo/Asia/Shanghai文件链接为localtime文件即可。

## 2.1 ntp程序

我们可以通过ntp程序自动校准系统时间。

安装ntp：`yum -y install ntp`

启动并设置开机自启：

- `systemctl start ntpd`
- `systemctl enable ntpd`

当ntpd启动后会定期的帮助我们联网校准系统的时间

- 也可以手动校准（<font color="red">需root权限</font>）：`ntpdate -u ntp.aliyun.com`

通过阿里云提供的服务网址配合ntpdate（安装ntp后会附带这个命令）命令自动校准。

## 3 IP地址、主机名

**IP地址是联网计算机的网络地址，用于在网络中进行定位和其他计算机进行通讯。**

IP地址主要有2个版本，V4版本和V6版本。

**IPV4版本的地址格式是：a.b.c.d，其中abcd表示0~255的数字**，如192.168.88.101就是一个标准的IP地址。

可以通过命令：ifconfig，查看本机的ip地址，如无法使用ifconfig命令，可以安装：`yum -y install net-tools`

![image-20230402144817267](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021448318.png)

### 3.1 特殊IP地址

除了标准的IP地址以外，还有几个特殊的IP地址需要我们了解：

- **127.0.0.1，本地回环IP，这个IP地址用于指代本机**
- **0.0.0.0**，特殊IP地址
  - 可以用于指代本机
  - 可以在端口绑定中用来确定绑定关系。
  - 在一些IP地址限制中，表示所有IP的意思，如放行规则设置为0.0.0.0，表示允许任意IP访问（在一些白名单中表示任意IP）。

### 3.2 主机名

每一台电脑除了对外联络地址（IP地址）以外，也可以有一个名字，称之为主机名，用于**标识**一个计算机。

无论是Windows或Linux系统，都可以给系统设置主机名。

- Windows系统主机名

![image-20230402145723600](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021457637.png)

- Linux系统主机名

![image-20230402151156383](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021511412.png)

### 3.3 在Linux中修改主机名

- 可以使用命令：hostname查看主机名

![image-20230402152342018](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021523056.png)

- 可以使用命令：`hostnamectl set-hostname 主机名`，修改主机名（需root）

![image-20230402161633495](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021616524.png)

- 重新登录shell即可看到主机名已经正确

![image-20230402162142433](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021621460.png)

### 3.4 域名解析（主机名映射）

IP地址实在是难以记忆，有没有什么办法可以通过主机名或替代的字符地址去代替数字化的IP地址呢？

实际上，我们一直都是通过字符化的地址去访问服务器，很少指定IP地址呢?

比如，我们在浏览器内打开：www.baidu.com，会打开百度的网址

其中，www.baidu.com，是百度的网址，我们称之为：域名。

> **可以通过主机名找到对应的IP地址，这就是主机名映射**。

**那么不是说通过IP地址才能访问服务器吗？**

**为什么域名这一串好记的字符也可以嘞？**

**这一切，都是域名解析帮我们解决的。**

> 比如打开浏览器访问百度网站，操作系统就会进行相应的检查，不同的系统，会去检查不同的相关文件中有没有记录百度和IP的对应关系。进行判断，如果有记录，则会直接打开网站；如果无记录，则联网查询公开的DNS服务器是否有记录www.baidu.com的IP地址。

![image-20230402192841352](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304021928458.png)

> 即：
>
> - 先查看本机的记录（私人地址本）
>   - Windows看：C:\Windows\System32\drivers\etc\hosts
>   - Linux看：/etc/hosts
> - 再联网去DNS服务器（如114.114.114.114，8.8.8.8等）询问



那么应该如何去自行地配置映射使得可以更加方便去访问呢？

### 3.5 配置主机名映射

比如shell是通过IP地址连接到的Linux服务器，那有没有可能通过域名（主机名）连接呢？

> 只需要在Windows系统的：C:\Windows\System32\drivers\etc\hosts文件中配置记录即可
![image-20230402205729780](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022219930.png)
![image-20230402210511021](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022220702.png)
> 打开shell的连接配置，即可用主机名进行连接。
![image-20230402210734439](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022107477.png)

### 3.6 虚拟机配置固定IP

#### 为什么需要固定IP？

当前我们虚拟机的Linux操作系统，其IP地址是通过**DHCP服务**获取的。

DHCP：动态获取IP地址，即每次重启设备后都会获取一次，可能导致IP地址频繁变更。

- 原因1：办公电脑IP地址变化无所谓，但是我们要远程连接到Linux系统，如果IP地址经常变化我们就要频繁修改适配很麻烦。
- 原因2：如果配置了虚拟机IP地址和主机名的映射，如果IP频繁更改，我们也需要频繁更新映射关系。

综上所述，我们需要IP地址固定下来，不要变化。

#### 在VMware Workstation中配置固定IP

在windows系统中配置固定IP需要**2个大步骤**：

1. +在VMware Workstation（或Fusion）中配置IP地址网关和网段（IP地址的范围）
2. 在Linux系统中手动修改配置文件，固定IP

**子网IP**：为了确定网络区域，分开主机和路由器的每个接口，从而产生了若干个分离的网络岛，接口端连接了这些独立网络的端点。这些独立的网络岛叫做子网(subnet)。IP地址是以网络号和主机号来表示网络上的主机的，只有在一个网络号下的计算机之间才能“直接”互通，不同网络号的计算机要通过网关（Gateway）才能互通。

> 配置固定IP详细步骤：
>
> 1. 打开VMware，打开编辑里的**虚拟网络编辑器**
>
>    ![image-20230402214034001](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022220769.png)
>
> 2. 选择VMnet8，配置子网IP以及子网掩码(子网掩码一定要确认是255.255.255.0)，然后打开NAT设置
>
>    ![image-20230402214341076](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022143132.png)
>
> 3. 在NAT设置中设置网关后点击确定。
>
>    ![image-20230402214723611](F:\桌面\image-20230402214723611.png)
>
> 4. 接下来需要在Linux系统中修改固定IP。
>
>    首先使用vim编辑 `/etc/sysconfig/network-scripts/ifcfg-ens33`文件，填入如下内容：
>
>    ![image-20230402220733717](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022221948.png)
>
> 5. 执行：systemctl restart network重启网卡，执行ifconfig即可看到ip地址固定为192.169.88.130了
>
>    ![image-20230402221806182](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304022218221.png)
>
> 记得更改shell中配置的ip地址哦~



