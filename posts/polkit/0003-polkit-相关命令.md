---
title: "polkit 相关命令"
date: 2022-06-07T19:55:26+08:00
tags: ['linux', 'polkit']
categories: ['linux']
draft: false
---

## pkcheck

> 检查进程是否被授权

### 说明
pkcheck用于检查进程(由--process(见下文)或--system-bus-name指定)是否被授权执行操作。可以多次使用--detail选项传递有关操作的细节。如果--allow-user-interaction被传递，pkcheck在等待身份验证时阻塞。

调用pkcheck --list-temp将列出当前会话的所有临时授权，而pkcheck --revoke-temp将撤销当前会话的所有临时授权。

这个命令是对polkit D-Bus接口的简单包装;详细信息请参见D-Bus接口文档。

```
Usage:
  pkcheck [OPTION...]

Help Options:
  -h, --help                         Show help options

Application Options:
  -a, --action-id=ACTION             Check authorization to perform ACTION
  -u, --allow-user-interaction       Interact with the user if necessary
  -d, --details=KEY VALUE            Add (KEY, VALUE) to information about the action
  --enable-internal-agent            Use an internal authentication agent if necessary
  --list-temp                        List temporary authorizations for current session
  -p, --process=PID[,START_TIME,UID] Check authorization of specified process
  --revoke-temp                      Revoke all temporary authorizations for current session
  -s, --system-bus-name=BUS_NAME     Check authorization of owner of BUS_NAME
  --version                          Show version

Report bugs to: http://lists.freedesktop.org/mailman/listinfo/polkit-devel
polkit home page: <http://www.freedesktop.org/wiki/Software/polkit>

```

### 返回值
如果指定的进程被授权，pkcheck将以0的返回值退出。如果授权结果包含任何详细信息，它们将使用环境样式的报告作为键/值对打印在标准输出上，例如，首先键后跟一个等号，然后值后跟一个换行符。
```
KEY1=VALUE1
KEY2=VALUE2
KEY3=VALUE3
...
```

不包含在[a-zA-Z0-9_]中的八进制代码使用以\为前缀的八进制代码进行转义。例如,utf-8字符串`flø你好将` 将显示为: `\303\270l\54\344\275\240\345\245\275`。

如果指定的进程未被授权，pkcheck将退出，返回值为1，并在标准错误时打印诊断消息。详细信息打印在标准输出上。

如果指定的进程没有获得授权，因为没有合适的身份验证代理可用，或者没有通过 --allow-user-interaction，那么pkcheck将退出，返回值为2，并在标准错误时打印一条诊断消息。详细信息打印在标准输出上。

如果指定的进程没有被授权，因为身份验证对话框/请求被用户驳回，pkcheck退出，返回值为3，并在标准错误上打印诊断消息。详细信息打印在标准输出上。

如果在检查授权时发生错误，pkcheck退出，返回值为127，并在标准错误上打印诊断消息。

如果传递的一个或多个选项是不规范的，则pkcheck退出，返回值为126。如果stdin是一个tty，那么也会显示此手册页。

### 注意
不要针对 --process 选项仅仅使用pid或pid,start-time 语法。新代码应该总是使用pid,pid-start-time,uid。start-time的值可以通过参考proc(5)文件系统来确定，具体取决于操作系统。如果传入的参数少于3个，pkcheck将尝试在内部查找它们，但注意这可能是不正常的。

## pkaction
> 获取已注册操作的详细信息

### 说明
pkaction用于获取关于已注册polkit操作的信息。如果调用时没有使用--action-id，则显示所有操作。如果调用时不带--verbose选项，则只显示操作的名称。否则将显示有关操作的详细信息。

```
Usage:
  pkaction [OPTION?] [--action-id ACTION]

Help Options:
  -h, --help                 Show help options

Application Options:
  -a, --action-id=ACTION     Only output information about ACTION
  -v, --verbose              Output detailed action information
  --version                  Show version

Report bugs to: http://lists.freedesktop.org/mailman/listinfo/polkit-devel
polkit home page: <http://www.freedesktop.org/wiki/Software/polkit>

```

### 返回值
如果成功，pkaction返回0。否则将返回非零值，并在标准错误时打印诊断消息。

## pkexec
> 以其他用户身份执行命令

### 说明
pkexec允许授权用户作为另一个用户执行PROGRAM。如果没有指定PROGRAM，将运行默认的shell。如果没有指定用户名，那么程序将以管理超级用户root执行。

### 返回值
成功完成后，返回值就是PROGRAM的返回值。如果调用进程未被授权或无法通过身份验证获得授权或发生错误，则pkexec退出，返回值为127。如果由于用户取消了身份验证对话框而无法获得授权，则pkexec退出，返回值为126。

### 认证代理

与任何其他polkit应用程序一样，pkexec将使用为调用进程或会话注册的身份验证代理。但是，如果没有可用的身份验证代理，那么pkexec将注册它自己的文本身份验证代理。可以通过传递--disable-internal-agent选项来关闭此行为。

### 安全方面
作为另一个用户执行程序是一种特权操作。默认情况下，要检查的操作(请参阅“action AND AUTHORIZATIONS”一节)需要管理员身份验证。此外，呈现给用户的身份验证对话框将显示要执行的程序的完整路径，以便用户知道将发生什么。

PROGRAM运行它的环境将被设置为最小已知和安全的环境，以避免通过`LD_LIBRARY_PATH`或类似机制注入代码。另外，`PKEXEC_UID`环境变量被设置为调用`pkexec`的进程的用户id。因此，pkexec默认情况下不允许您以另一个用户的身份运行X11应用程序，因为没有设置`$DISPLAY`和`$XAUTHORITY`环境变量。如果action上的`org.freedesktop.policykit.exec.allow_gui`注释被设置为非空值，那么这两个变量将被保留;但是，这是不鼓励的，应该只用于遗留程序。

注意，pkexec不验证传递给`PROGRAM`的`ARGUMENTS`。在正常情况下(每次使用pkexec时都需要管理员身份验证)，这不是问题，因为如果用户是管理员，他可能只需要运行pkexec bash来获得根用户。

但是，如果使用了用户可以保留授权的操作(或者用户是隐式授权的)，这可能是一个安全漏洞。因此，根据经验，更改默认所需授权的程序永远不应该隐式信任用户输入(例如，像任何其他编写良好的suid程序)。

### action和授权
默认使用“`org.freedesktop.policykit.exec`”动作。要使用另一个操作，请在一个action上使用`org.freedesktop.policykit.exec.path`注释，该注释的值设置为程序的完整路径。除了指定程序之外，还可以指定身份验证消息、描述、图标和默认值。如果有`org.freedesktop.policykitexec.argv1`注释，则只有当程序的第一个参数与注释的值匹配时，才会选择该操作。

请注意，身份验证消息可能会引用变量(请参阅“变量”一节)，例如$(user)将被扩展为用户变量的值。

### 包装使用
为了避免修改现有软件以使用pkexec作为命令行调用的前缀，可以在she-bang包装器中使用pkexec，如下所示:

```python
#!/usr/bin/pkexec /usr/bin/python

import os
import sys

print "Hello, I'm running as uid %d"%(os.getuid())

for n in range(len(sys.argv)):
    print "arg[%d]=`%s'"%(n, sys.argv[n])
```

如果这个脚本被安装到/usr/bin/my-pk-test，那么下面的注释
```
[...]
<annotate key="org.freedesktop.policykit.exec.path">/usr/bin/python</annotate>
<annotate key="org.freedesktop.policykit.exec.argv1">/usr/bin/my-pk-test</annotate>
[...]

```

可以用来选择合适的polkit动作。注意正确使用后面的注释，否则它将匹配`/usr/bin/python`脚本的任何pkexec调用。

### 变量
以下变量是由pkexec设置的。它们可以用于授权规则和身份验证对话框中显示的消息:
|参数|说明|
|:---|:---|
|`program`|要执行程序的完全限定路径。例如:“/bin/cat”|
|`command_line`|请求的命令行(不要将其用于任何安全检查，它不安全)。示例:“cat /srv/xyz/foobar”|
|`user`|执行程序的用户的用户名|
|`user.gecos`|执行程序的用户的全名|
|`user.display`|用户执行程序的一种表示形式，适合在身份验证对话框中显示。通常设置为用户名和全名的组合。例如:“David Zeuthen (davidz)”|


## pkttyagent
> 文本验证助手

### 说明
pkttyagent用于启动由 --process 或 --system-bus-name指定的主题的文本身份验证代理。如果没有提供这两个选项，则使用父进程。

要在注册身份验证代理时获得通知，可以侦听changed D-Bus信号，或使用--notify-fd传递已传递给程序的文件描述符的编号。当身份验证代理成功注册后，将关闭此文件描述符。

如果使用 --fallback，文本身份验证代理将不会替换现有的身份验证代理。

```
Usage:
  pkttyagent [OPTION?]

Help Options:
  -h, --help                         Show help options

Application Options:
  --fallback                         Don't replace existing agent if any
  --notify-fd=FD                     Close FD when the agent is registered
  -p, --process=PID[,START_TIME]     Register the agent for the specified process
  -s, --system-bus-name=BUS_NAME     Register the agent for the owner of BUS_NAME
  --version                          Show version

Report bugs to: http://lists.freedesktop.org/mailman/listinfo/polkit-devel
polkit home page: <http://www.freedesktop.org/wiki/Software/polkit>

```

### 返回值
如果无法注册身份验证代理，pkttyagent将退出，退出码为127。诊断消息在标准错误时打印。

如果传递的一个或多个选项是不正确的，pkttyagent将退出，退出代码为126。如果stdin是一个tty，那么也会显示此手册页。

如果成功注册了身份验证代理，pkttyagent将继续运行，根据需要与用户交互。当不再需要它的服务时，可以终止该进程。

### 注意
因为进程标识符可以循环使用，所以在使用 --process选项时，调用者应该始终使用pid,pid-start-time。pid-start-time的值可以通过参考proc(5)文件系统来确定，具体取决于操作系统。如果只将pid传递给--process选项，那么pkttyagent将查找启动时间本身，但请注意，这可能是不正常的。

