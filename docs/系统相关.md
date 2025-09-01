# Windows

## 软件推荐

| 软件名                 | 描述    |
|:------------------- | ----- |
| Geek                | 卸载    |
| Dism++              | 系统工具  |
| WiseRegisterCleaner | 注册表清理 |
| Bandzip             | 解压工具  |
| 拾光                  | 优质壁纸  |
| Wiztree             | 存储分析  |
| Clash               | 代理    |
| Win11轻松设置           | 优化    |
| Start11             | 开始菜单  |
| FirePE\|WePE        | PE工具  |
| HEU_KMS             | 激活工具  |



## Vmware共享文件夹

需要先**安装vmtools**

打开虚拟机设置，启用共享文件夹

虚拟机访问/mnt/hgfs/即可

---

# Ubuntu

## Ubuntu清理

通过一下命令查看磁盘使用率：

```shell
df -h
```

移除不再需要的软件包：

```shell
sudo apt-get autoremove --purge
```

移除软件包缓存：

```shell
sudo apt-get clean
```

清理日志：

```shell
sudo journalctl --vacuum-time=2weeks
```

清理临时文件：

```shell
sudo rm -rf /tmp/*
```

> 清理旧内核(请确定清理的已经**不再使用**的内核)：
> 
> ```shell
> sudo apt-get remove --purge linux-image-x.x.x-xx-generic
> ```



## Ubuntu换源

### 方法一

1. 进入 系统设置
2. 打开 软件和更新
3. 设置 下载自… 其他站点
4. 通过 选择最佳服务器 选择速度最快的镜像源
   
   

### 方法二

1. 备份镜像设置文件（可选）：

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

2. 编辑镜像源文件：

```shell
sudo vi /etc/apt/sources.list
```

3. 搜索镜像源，并写入文件

4. 执行命令：

```shell
sudo apt-get update
sudo apt-get upgrade
```



## C/C++环境配置

输入命令：

```shell
sudo apt-get install gcc
sudo apt-get install build-essential
```



## 常见报错

### 可执行程序存在却报错:bash no such file or directory

使用：

```shell
file [文件名]
```

查看文件信息，如果机器为64位系统而程序为32位程序，缺少运行库：

```shell
sudo apt-get install lib32stdc++6
```



---

# WSL

## Linux子系统--WSL安装

1. 开启Linux子系统和Hype-v 

> 控制面板-->程序-->启用或关闭WIndows功能-->适用于Windows的Linux子系统&&Hype-v

2. 设置WSL版本为WSL2

> - 打开终端
> 
> - 执行命令
> 
> ```bash
> wsl --set-default-version 2
> ```

3. 在Microsoft store中安装以Linux发行版

> 启动即可



### WSL1与WSL2转换

将 WSL 1 上的 Ubuntu 转换到 WSL 2
如果使用 WSL 1，则可以将现有的 WSL 1 安装升级到 WSL 2。要将现有的 WSL 1 发行版转换到 WSL 2，请在 PowerShell 中运行以下命令：

```bash
wsl.exe--set-versionUbuntu2
```

> 使用时，应将命令中的 “Ubuntu” 替换为您在 WSL 1 上安装运行的对应发行版的名称



### 常见问题

1. 安装失败并出现错误**0x80070003**
   适用于 Linux 的 Windows 子系统只能在系统驱动器（通常是 C: 驱动器）中运行。 请确保分发版存储在系统驱动器上：打开“设置”->“系统”-->“存储”-> “更多存储设置”： 更改新内容的保存位置”

2. WslRegisterDistribution 失败并出现错误 **0x8007019e**
   未启用“适用于 Linux 的 Windows 子系统”可选组件：
   打开“控制面板” -> “程序和功能” -> “打开或关闭 Windows 功能”-> 选中“适用于 Linux 的 Windows 子系统”，或使用本文开头所述的 PowerShell cmdlet。

3. 安装失败，出现错误 **0x80070003** 或错误 **0x80370102**
   请确保在计算机的 BIOS 内已启用虚拟化。 有关如何执行此操作的说明因计算机而异，并且很可能在 CPU 相关选项下。WSL2 要求 CPU 支持二级地址转换 (SLAT) 功能，后者已在 Intel Nehalem 处理器（Intel Core 第一代）和 AMD Opteron 中引入。 即使成功安装了虚拟机平台，旧版 CPU（例如 Intel Core 2 Duo）也无法运行 WSL2。

4. 尝试升级时出错：
   
   ```bash
   Invalid command line option: wsl --set-version Ubuntu 2
   ```
   
   请确保已启用适用于 Linux 的 Windows 子系统，并且你使用的是 Windows 内部版本 18362 或更高版本。 若要启用 WSL，请在 PowerShell 提示符下以具有管理员权限的身份运行此命令：
   
   ```bash
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
   ```

5. 由于虚拟磁盘系统的某个限制，无法完成所请求的操作。虚拟硬盘文件必须是解压缩的且未加密的，并且不能是稀疏的
   
   - 取消选中“压缩内容”（如果已选中“加密内容”，请一并取消选中），方法是打开 Linux 发行版的配置文件文件夹。 它应位于 Windows 文件系统上的一个文件夹中，类似于：USERPROFILE%\AppData\Local\Packages\CanonicalGroupLimited...
   
   - 在此 Linux 发行版配置文件中，应存在一个 LocalState 文件夹。 右键单击此文件夹可显示选项的菜单。 选择“属性”>“高级”，然后确保未选择（未勾选）“压缩内容以节省磁盘空间”和“加密内容以保护数据”复选框。 如果系统询问是要将此应用到当前文件夹还是应用到所有子文件夹和文件，请选择“仅此文件夹”，因为你只是要清除压缩标志。 完成此操作后，wsl --set-version 命令应正常工作。

6. 无法将词语“wsl”识别为 cmdlet、函数、脚本文件或可运行程序的名称
   请确保已安装“适用于 Linux 的 Windows 子系统”可选组件。 此外，如果你使用的是 ARM64 设备，并从 PowerShell 运行此命令，则会收到此错误。 请改为从 PowerShell Core 或从命令提示符运行 wsl.exe。

7. 错误：此更新仅适用于装有适用于 Linux 的 Windows 子系统的计算机
   
   - 若要安装 Linux 内核更新 MSI 包，需要 WSL，应先启用它。 如果失败，将看到以下消息：This update only applies to machines with the Windows Subsystem for Linux。
   
   - 出现此消息有三个可能的原因：

8. 你仍使用旧版 Windows，不支持 WSL 2。 有关版本要求和要更新的链接，请参阅步骤 #2。

9. 未启用 WSL。 需要返回到步骤 #1，并确保在计算机上启用了可选的 WSL 功能。

10. 错误：WSL 2 要求对其内核组件进行更新
    如果 %SystemRoot%\system32\lxss\tools 文件夹中缺少 Linux 内核包，会遇到此错误。 若要解决此问题，请在安装说明的步骤 #4 中安装 Linux 内核更新 MSI 包。 可能会需要从添加或删除程序卸载 MSI，然后重新安装。
    
    

## WSL迁移到D盘

1. 查看自己的wsl和ubuntu版本；
   
   ```bash
   wsl -l -v
   ```

2. 关闭wsl服务；
   
   ```bash
   wsl --shutdown
   ```

3. 将原位置的ubuntu导出到指定位置(我的是E盘)；
   
   ```bash
   wsl --export Ubuntu D:\Ubuntu.tar
   ```

4. 原wsl注销ubuntu；
   
   ```bash
   wsl --unregister Ubuntu
   ```

5. 在指定位置(我的是E盘)导入ubuntu；
   
   ```bash
   wsl --import Ubuntu D:\WSL D:\Ubuntu.tar
   ```

6. 修改用户名为原来的名字。
   
   ```bash
   Ubuntu config --default-user ysh
   ```
   
   

## 减少WSL2在磁盘上占用的空间

WSL2 使用了虚拟磁盘，意味着它可能只有 15GB 的数据，但是虚拟磁盘占用了 100GB 的空间。如果你往 WSL2 中放了大量的数据，然后就删掉，会发现WSL2的磁盘占用没有降下来，这就是虚拟磁盘造成的。

1. 准备工作
   在压缩虚拟磁盘前，需要将 WSL2 先关闭。可以先使用命令行来检查它的状态：

```bash
wsl.exe --list --verbose
```

        如果没有关闭（状态是 Running ），再用命令行去关闭它：

```bash
wsl.exe --terminate
```

> 建议先备份一下WSL2的数据。

2. 使用diskpart压缩虚拟磁盘
   在命令行启动 diskpart 工具，它会自己打开一个新的窗口：

```bash
diskpart
```

        接下来需要确定虚拟磁盘文件的位置。

> WSL2的虚拟磁盘文件在C:\Users\{user}\AppData\Local\Packages\下面，不同的WSL2发行版对应的名称不同，例如 Pengwin 是 > WhitewaterFoundryLtd.Co，Ubuntu 是 CanonicalGroupLimited，Debian 是TheDebianProject 。找到了WSL2 的文件夹，就能在它下面找到 > LocalState\ **ext4.vhdx**这个磁盘文件。

        用 diskpart 选择这个文件：

```bash
select vdisk file = "{虚拟磁盘路径}"
```

        再执行压缩命令：

```bash
compact vdisk
```

3. 执行成功
   
   
   
   

## WSL配置代理

1. 终端输入命令：

```shell
vim ~/.bashrc
```

2. 在文末输入以下内容：

```bash
        host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
        export http_proxy="http://$host_ip:7890"    # 端口根据代理软件来确定
        export https_proxy="http://$host_ip:7890"
```

3. 退出保存
   
   

> 更简单的方式：使用代理软件的**Tun模式**（该模式会虚拟一张网卡，所有系统流量都会由代理软件接管）



## WSL配置服务自启动

以ssh服务为例：

创建文件：

```shell
sudo vim /etc/init.wsl
```

写入启动服务信息：

```shell
/etc/init.d/ssh $1
```

授予文件可执行权限

```shell
sudo chmod +x /etc/init.wsl
```

测试执行权限：

```shell
sudo /etc/init.wsl start
```

**创建启动脚本：** 在你的 WSL 中创建一个脚本，比如 `start.sh`，内容如下：

```shell
sh -c "echo '666' | sudo -S /etc/init.wsl start"
```

> echo '[root的用户密码]'

授予脚本可执行权限：

```shell
sudo chmod +x start.sh
```

**在 WSL 启动时运行脚本：** 你可以将该脚本放在 `.bashrc` 或 `.bash_profile` 中：

```shell
echo 'bash ~/start.sh' >> ~/.bashrc
```

测试



### SSH远程连接WSL

安装ssh服务：

```shell
sudo apt-get install openssh-serer
```

修改配置信息``/etc/ssh/sshd_config``：

```shell
Port 22 #端口
PermitRootLogin yes #允许远程root用户登录
PasswordAuthentication yes #允许使用用户名密码登录
```

重启ssh服务：

```shell
sudo service ssh restart
```



本机Windows连接测试（需要安装ssh功能：设置-->系统-->可选功能-->添加可选功能--OpenSSH客户端）：

```bash
ssh root@localhost [-p 22]
```



**此时并不能远程连接WSL，需要配置端口转发和防火墙**



查看WSL的ip地址(此步可省略)：

```shell
ifconfig
```

> eth0下的inen4

设置端口转发（使用IPV6转发）：

```bash
netsh interface portproxy add v6tov4 listenaddress=:: listenport=22 connectaddress=localhost connectport=22
```

> 转发IPV4
> 
> ```bash
> netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=localhost connectport=22
> ```
> 
> 注意：
> 
> connectaddress应填写WSL中的IP地址，经测试可用localhost代替
> 
> 端口可自行更改

查看端口是否已经被监听：

```bash
netstat -ano | findstr :22
```

> 如果没有返回内容，请检查：
> 
> 系统的Ip Helper服务是否开启（进入服务管理查看）
> 
> 确认vEthernet是否开启IPV6

防火墙添加端口22（实际设置的端口）的入站规则；

使用IPV6连接测试



其他命令：

显示配置的所有端口转发:

```bash
netsh interface portproxy show all
```

重置端口转发：

```bash
netsh interface portproxy reset
```
