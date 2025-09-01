# Linux操作

## 远程登录（telnet、SSH(Putty)、ftp、远程桌面(VNC)）

> 启动服务，配置防火墙

```shell
telnet <IP>  # 使用Telnet登录远程主机（不推荐，不安全）

ssh <用户名>@<IP> [-p port]  # 使用SSH登录远程主机，-p指定端口

ftp <IP>  # 使用FTP登录远程主机，需要启动vsftpd服务
```

### SSH 常用选项
```shell
ssh -i <私钥文件> <用户名>@<IP>  # 使用私钥文件进行身份验证
ssh -L <本地端口>:<远程主机>:<远程端口> <用户名>@<IP>  # 本地端口转发
ssh -R <远程端口>:<本地主机>:<本地端口> <用户名>@<IP>  # 远程端口转发
```

### FTP 常用命令
```shell
ftp <IP>
> get <文件名>  # 下载文件
> put <文件名>  # 上传文件
> mget <文件名>  # 下载多个文件
> mput <文件名>  # 上传多个文件
> bye  # 退出FTP
```

---

## 指令（vim、正则表达式）

> ctrl+C    终止程序  
> ctrl+D    结束输入

```shell
cat -A <file>  # -A显示特殊字符

ls -a  # 列出所有文件，包括隐藏文件

ls -l  # 显示文件详细信息

ls -al  # 以上组合

cp [-i] [-f] [-r]  # 复制文件或目录，-i交互式提示，-f强制覆盖，-r递归复制

mv [源] [目的]  # 移动文件或目录，也可用于重命名

rm [-i] [-f] [-r]  # 删除文件或目录，-i交互式提示，-f强制删除，-r递归删除

|  # 管道，前一条指令的标准输出作为下一条指令的标准输入

grep  # 匹配字符串，-n显示行号，-c计数，-v反选
      # 配合正则表达式 [菜鸟](https://www.runoob.com/regexp/regexp-syntax.html)
```

### 常用正则表达式示例
```shell
grep '^start' file  # 匹配以"start"开头的行
grep 'end$' file  # 匹配以"end"结尾的行
grep 'a.b' file  # 匹配"a"和"b"之间有一个任意字符的字符串
grep 'a.*b' file  # 匹配"a"和"b"之间有任意数量字符的字符串
```

---

## Vim 编辑器

### 普通模式（输入模式按下ESC）
```shell
dd  # 删除当前行
dn  # 删除当前行以下n行
u  # 撤销
p  # 粘贴
```

### 命令模式（输入模式按下ESC并输入英文字符":"）
```shell
i  # 切换到输入模式，在光标当前位置开始输入文本
x  # 删除当前光标所在处的字符
:  # 切换到底线命令模式，以在最底一行输入命令
a  # 进入插入模式，在光标下一个位置开始输入文本
o  # 在当前行的下方插入一个新行，并进入插入模式
O  # 在当前行的上方插入一个新行，并进入插入模式
dd  # 剪切当前行
yy  # 复制当前行
p  # 粘贴剪贴板内容到光标下方
P  # 粘贴剪贴板内容到光标上方
u  # 撤销上一次操作
Ctrl + r  # 重做上一次撤销的操作
:w  # 保存文件
:q  # 退出 Vim 编辑器
:q!  # 强制退出Vim 编辑器，不保存修改
```

### Vim 高级操作
```shell
:set number  # 显示行号
:set nonumber  # 隐藏行号
:split <文件名>  # 水平分割窗口并打开新文件
:vsplit <文件名>  # 垂直分割窗口并打开新文件
:e <文件名>  # 打开新文件
:wq  # 保存并退出
```

---

## Shell 输入/输出重定向

| 命令            | 说明                                             |
| --------------- | ------------------------------------------------ |
| command > file  | 将输出重定向到 file                              |
| command < file  | 将输入重定向到 file                              |
| command >> file | 将输出以追加的方式重定向到 file                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file |
| n >& m          | 将输出文件 m 和 n 合并                           |
| n <& m          | 将输入文件 m 和 n 合并                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入 |

### 高级重定向操作
```shell
command 2> file  # 将标准错误重定向到文件
command &> file  # 将标准输出和标准错误都重定向到文件
command > file 2>&1  # 将标准输出和标准错误都重定向到文件
command < file1 > file2  # 从file1读取输入，将输出写入file2
```

---

## 打包解包/压缩解压

```shell
bzip2  # 压缩解压，后缀.bz2
gzip  # 压缩解压，后缀.gz
tar  # 打包解包，后缀.tar
```

### 常用命令
```shell
tar -xvf file.tar  # 解包tar文件
tar -xvzf file.tar.gz  # 解包并解压gzip压缩的tar文件
tar -xvjf file.tar.bz2  # 解包并解压bzip2压缩的tar文件
zip -r archive.zip folder  # 压缩文件夹为zip文件
unzip archive.zip  # 解压zip文件
```

---

## 多用户间通信

### write 命令
```shell
write <用户名>  # 开始向指定用户发送消息
Ctrl+D  # 结束消息发送
```

### mail 命令
```shell
mail <用户名>  # 发送邮件给指定用户
Subject: <主题>  # 输入邮件主题
<邮件内容>  # 输入邮件内容
Ctrl+D  # 结束邮件输入
```

---

## 文件链接

```shell
ln [参数][源文件或目录][目标文件或目录]
```

### 软链接
- 以路径的形式存在，类似于Windows中的快捷方式。
- 可以跨文件系统。
- 可以对不存在的文件名或目录进行链接。

### 硬链接
- 以文件副本的形式存在，但不占用实际空间。
- 不允许给目录创建硬链接。
- 只能在同一个文件系统中创建。

### 常用参数
```shell
-s  # 创建软链接
-v  # 显示详细的处理过程
```

> 建议使用**绝对路径**创建文件链接，避免移动文件后链接失效。

---

## 后台任务

```shell
[command] &  # 将命令放到后台执行

jobs  # 查看后台任务

kill %[jobs_id]  # 结束后台任务
```

---

## C程序编译、makefile编写

### 编译C程序
```shell
gcc -o output_file source_file.c  # 编译C程序
```

### Makefile 示例
```makefile
CC = gcc
CFLAGS = -Wall -g

all: program

program: main.o utils.o
    $(CC) $(CFLAGS) -o program main.o utils.o

main.o: main.c
    $(CC) $(CFLAGS) -c main.c

utils.o: utils.c
    $(CC) $(CFLAGS) -c utils.c

clean:
    rm -f *.o program
```

