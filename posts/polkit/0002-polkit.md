---
title: "polkit 授权管理器"
date: 2022-06-07T18:55:26+08:00
tags: ['linux', 'polkit', 'Authorization manager']
categories: ['linux']
draft: false
---

## 说明

polkit提供了一个授权API，特权程序("MECHANISMS")使用它为非特权程序("SUBJECTS")提供服务，这些服务通常通过某种进程间通信机制实现。在这种情况下，特权程序通常将非特权程序视为不可信的。对于来自某个非特权程序的每个请求，该特权程序需要确定该请求是被授权的，还是应该拒绝为该非特权程序服务。使用polkit api，一种机制可以将此决策转移给受信任的一方:polkit。

<br/>

polkit是作为系统守护进程polkitd(8)实现的，它本身没有什么特权，因为它是作为polkitd系统用户运行的。机制、主体和身份验证代理使用系统消息总线与权威机构通信。

<br/>

除了充当授权机构之外，polkit还允许用户通过对管理用户或客户端所属会话的所有者进行身份验证来获得临时授权。这对于需要验证系统操作员是否真的是用户或管理用户的机制非常有用。

### 系统架构

polkit的系统体系结构由Authority(作为系统消息总线上的服务实现)和每个用户会话(由用户的图形环境提供和启动)的Authentication Agent组成。操作由应用程序定义。厂商、站点和系统管理员可以通过“授权规则”控制授权策略。

<div align=center><img src='/pic/polkit/1.png'/></div>
<center>polkit 系统架构</center>

为了方便起见，libpolkit-gobject-1库封装了polkit D-Bus API，可用于任何C/C++程序以及支持GObjectIntrospection的高级语言，如JavaScript和Python。一种机制也可以使用D-Bus API或pkcheck(1)命令来检查授权。libpolkit-agent-1库提供了一个本地认证系统的抽象，例如pam(8)，还提供了注册和与polkit D-Bus服务通信的设施。

### 认证代理
身份验证代理用于使会话的用户证明该会话的用户确实是该用户(通过身份验证为用户)或管理用户(通过身份验证为管理员)。为了与用户会话的其他部分很好地集成(例如匹配外观和感觉)，身份验证代理应该由用户使用的用户会话提供。例如，身份验证代理可能看起来像这样:

<div align=center><img src='/pic/polkit/2.png'/></div>

如果系统配置为没有root帐户，它可能会提示指定一个特定的用户作为管理用户:

<div align=center><img src='/pic/polkit/3.png'/></div>

不运行在桌面环境下的应用程序(例如，如果从ssh(1)登录启动)可能没有与它们相关联的身份验证代理。这样的应用程序可以使用PolkitAgentTextListener类型或pkttyagent(1)助手，这样用户可以使用文本接口进行身份验证。

### 声明的`Action`

特权程序需要声明一组`actions`才能使用polkit。`action`对应于客户机可以请求特权程序执行的操作，并定义在XML文件中，特权程序将这些文件安装到/usr/share/polkit-1/actions目录中。

polkit操作有名称空间，只能包含字符“[A-Z][a-z][0-9].-”，即: ASCII、数字、句点和连字符。每个XML文件可以包含多个操作，但是所有操作都需要在同一个名称空间中，并且文件需要以名称空间命名并具有.policy扩展名。

XML文件必须具有以下doctype声明
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE policyconfig PUBLIC "-//freedesktop//DTD polkit Policy Configuration 1.0//EN"
"http://www.freedesktop.org/software/polkit/policyconfig-1.dtd">
```

policyconfig元素必须只出现一次。可以在policyconfig中使用的元素包括:
|字段|说明|
|:---|:---|
|`vendor`|在XML文档中提供操作的项目或供应商的名称。可选的|
|`vendor_url`|指向在XML文档中提供操作的项目或供应商的URL。可选的|
|`icon_name`|表示在XML文档中提供操作的项目或供应商的图标。图标名称必须遵循Freedesktop.org图标命名规范。可选的|
|`action`|声明了一个action。操作名称使用id属性指定，并且只能包含字符“[A-Z][A-Z][0-9].-”，即: ASCII、数字、句点和连字符|

可以在内部操作中使用的元素包括:
|字段|说明|
|:---|:---|
|`description`|对操作的可读描述|
|`message`|当用户需要身份验证时要求凭证时，显示给用户的提示消息，说明为什么要做权限验证|
|`defaults`|此元素用于为客户端指定隐式授权。可以在默认值中使用的元素包括:<br/>`allow_any`: 适用于任何客户端的隐式授权。可选的。<br/>`allow_inactive`: 隐式授权，应用于本地控制台上非活动会话中的客户端。可选<br/> `allow_active`: 应用于本地控制台活动会话中的客户端的隐式授权。可选的。<br/><br/>其中`allow_any`, `allow_inactive`和`allow_active`元素可以包含以下值:<br/>`no`:未验证通过<br/>`yes`:验证通过<br/>`auth_self`:需要由客户机所在会话的所有者进行身份验证。注意，对于多用户系统的大多数使用，这是不够严格的;一般推荐使用`auth_admin*`<br/>`auth_admin`:需要由管理用户进行身份验证<br/>`auth_self_keep`:类似于`auth_self`，但授权只保留一小段时间(例如5分钟)。上面关于`auth_self`的警告同样适用<br/>`auth_admin_keep`:同上|
|`annotate`|用于用键/值对注释操作。键是使用key属性指定的，值是使用value属性指定的。该元素可以出现0次或多次。见下面的已知注释。|
|`vendor`|用于在每个操作的基础上重写供应商。可选的|
|`vendor_url`|用于在每个操作的基础上重写供应商URL。可选的|
|`icon_name`|用于在每个操作的基础上重写图标名称。可选的|

对于本地化，`description`和`message`元素可以在不同的xml:lang属性中出现多次。
要列出已安装的polkit action，使用pkaction(1)命令。

### 已知的`annotations`

polkit附带的pkexec程序使用`org.freedesktop.policykit.exec.path` annotations 请参阅pkexec(1)手册页了解详细信息。

`org.freedesktop.policykit.imply` annotations(它的值是一个包含空格分隔的操作标识符列表的字符串)可以用来定义元操作。它的工作方式是，如果一个主题被授权执行带有该注释的操作，那么它也被授权执行该注释指定的任何操作。这个注释的典型用法是在定义带有单个锁定按钮的UI shell时，该按钮应该可以从不同的机制解锁多个操作。

`org.freedesktop.policykit.owner` annotations可用于定义一组用户，这些用户可以查询客户端是否被授权执行此操作。如果没有指定此注释，则只有根用户可以查询作为不同用户运行的客户端是否被授权执行某个操作。这个注释的值是一个字符串，包含一个空格分隔的PolkitIdentity条目列表，例如“unix-user:42 unix-user:colord”。这个注释的典型用途是作为系统用户而不是根用户运行的守护进程。

## 验证规则

polkitd从 `/etc/polkit-1/rules.d` 和 `/usr/share/polkit-1/rules.d` 中读取 `.rules`， 方法是根据每个文件的基名按词法顺序对文件进行排序(如果有一个tie， /etc中的文件会在/usr中的文件之前处理)。例如，对于以下四个文件，顺序是
- /etc/polkit-1/rules.d/10-auth.rules
- /usr/share/polkit-1/rules.d/10-auth.rules
- /etc/polkit-1/rules.d/15-auth.rules
- /usr/share/polkit-1/rules.d/20-auth.rules

这两个目录都受到监控，因此，如果更改、添加或删除规则文件，则清除现有的规则，并再次读取和处理所有文件。规则文件是用JavaScript编程语言编写的，并通过全局polkit对象(类型为polkit)与polkitd接口。

虽然在polkit的特定版本中使用的JavaScript解释器可能支持非标准特性(比如let关键字)，但授权规则必须符合ECMA-262 edition 5(换句话说，在polkit的未来版本中使用的JavaScript解释器可能会改变)。

授权规则针对两个特定的受众:
- 系统管理员
- 特殊用途的操作系统/环境
而且只针对这些观众。特别是，应用程序、机制和通用操作系统绝不能包含任何授权规则。

### The Polkit type

以下方法在polkit对象上可用:
```c
void addRule(polkit.Result function(action, subject) {...});
void addAdminRule(string[] function(action, subject) {...});
void log(string message);
string spawn(string[] argv);

```

`addRule()`方法用于添加一个函数，在执行动作和主题的授权检查时可以调用该函数。函数被调用的顺序是它们被添加的顺序，直到其中一个函数返回一个值。因此，要添加一个在其他规则之前处理的授权规则，请将其放在`/etc/polkit-1/rules.d`的文件中，其名称排在其他规则文件之前，例如`00-early-checks.rules`。每个函数都应该从polkit.Result返回一个值
```c
polkit.Result = {
    NO              : "no",
    YES             : "yes",
    AUTH_SELF       : "auth_self",
    AUTH_SELF_KEEP  : "auth_self_keep",
    AUTH_ADMIN      : "auth_admin",
    AUTH_ADMIN_KEEP : "auth_admin_keep",
    NOT_HANDLED     : null
};
```

对应于可以用作默认值的值。如果函数返回`polkit.Result.NOT_HANDLED`、`null`、`undefined`或`不返回值`，将尝试下一个用户函数。

记住，如果返回`polkit.result.AUTH_SELF_KEEP`或`polkit.Result.AUTH_ADMIN_KEEP`，那么在下一个短时间内(例如5分钟)，对相同动作标识符和主题的授权检查将会成功(即返回`polkit.Result.YES`)，即使检查传递的变量不同。因此，如果授权规则的结果依赖于这些变量，那么它不应该使用“`*_KEEP`”常量(如果需要类似的功能，那么授权规则可以使用时间戳的Date类型轻松实现临时授权)。

`addAdminRule()`方法用于添加一个函数，在需要进行管理员身份验证时可以调用该函数。该函数用于指定哪些身份可以用于管理员身份验证，以便进行由动作和主题标识的授权检查。添加的函数将按照添加的顺序调用，直到其中一个函数返回值为止。每个函数都应该返回一个字符串数组，其中每个字符串的形式为"unix-group:<group>"， "unix-netgroup:<netgroup>"或"unix-user:<user>"。如果函数返回null、未定义或根本不返回值，则尝试下一个函数。

不能保证使用`addRule()`或`addAdminRule()`注册的函数会被调用——例如，早期的规则文件可以注册一个总是返回值的函数，从而确保以后添加的函数永远不会被调用。

如果用户提供的代码执行时间较长，则会抛出异常，通常会导致函数终止(当前限制为15秒)。这用于捕获失控脚本。

spawn()方法生成一个由参数向量argv标识的外部helper，并等待它终止。如果发生错误或helper没有以退出码0正常退出，则会抛出异常。如果助手没有在10秒内退出，它就会被杀死。否则，程序的标准输出将作为字符串返回。应该谨慎使用spawn()方法，因为helper可能需要很长时间或不确定的时间来完成，并且在helper运行时不能处理其他授权检查。注意，生成的程序将作为无特权的polkitd系统用户运行。

`log()`方法将给定的消息写入以JavaScript文件名和行号为前缀的系统日志记录器。日志条目是使用`LOG_AUTHPRIV`标志发出的，这意味着日志条目通常最后出现在文件`/var/log/secure`中。log()方法通常只在调试规则时使用。Action和Subject类型有适合于方便日志记录的toString()方法，例如，
```javascript
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.policykit.exec") {
        polkit.log("action=" + action);
        polkit.log("subject=" + subject);
    }
});
```

当用户在shell中运行'pkexec -u bateman bash -i'时将产生以下结果:
```
May 24 14:28:50 thinkpad polkitd[32217]: /etc/polkit-1/rules.d/10-test.rules:3: action=[Action id='org.freedesktop.policykit.exec' command_line='/usr/bin/bash -i' program='/usr/bin/bash' user='bateman' user.gecos='Patrick Bateman' user.display='Patrick Bateman (bateman)']
May 24 14:28:50 thinkpad polkitd[32217]: /etc/polkit-1/rules.d/10-test.rules:4: subject=[Subject pid=1352 user='davidz' groups=davidz,wheel, seat='seat0' session='1' local=true active=true]
```

### The Action type
传递给用户函数的action参数是一个对象，其中包含正在检查的操作的信息。它的类型是Action，具有以下属性:
- string id: 动作标识符，例如org.freedesktop.policykit.exec。
以下方法在Action类型上可用:
```
string lookup(string key);

```

lookup()方法用于查找从该机制传递的polkit变量。例如，pkexec(1)机制设置可以在JavaScript中使用表达式action.lookup("program")获得的变量程序。如果给定的键没有值，则返回undefined。

请参阅每种机制的文档，了解每个操作可以使用哪些变量。

### The Subject type

传递给用户函数的subject参数是一个带有正在检查的进程信息的对象。它的类型是Subject，具有以下属性
|类型|说明|
|:---|:---|
|int pid|进程id|
|string user|用户名|
|string[] groups|用户所属的组|
|string seat|subject所属的seat，不是本地seat则为空(ps:关于seat查看linux seat了解)|
|string session|subject所属session|
|boolean local|仅仅是本地seat才会设置为 `true`|
|boolean active|当session是active才会设置为`true`|
以下方法可用于Subject类型:
```js
boolean isInGroup(string groupName);
boolean isInNetGroup(string netGroupName);

```

isInGroup()方法可以用来检查subject是否在给定的组中，isInNetGroup()方法可以用来检查subject是否在给定的网络组中。

### Authorization Rules Examples

允许admin组的所有用户进行用户管理，不改变其他用户的策略:
```js
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.accounts.user-administration" &&
        subject.isInGroup("admin")) {
        return polkit.Result.YES;
    }
});
```

定义管理用户为wheel组中的用户:
```js
polkit.addAdminRule(function(action, subject) {
    return ["unix-group:wheel"];
});
```

禁止子组中的用户更改主机名配置(即任何标识符以org.freedesktop.hostname1开头的操作)，并允许其他任何人在验证为自己之后这样做:
```js
polkit.addRule(function(action, subject) {
    if (action.id.indexOf("org.freedesktop.hostname1.") == 0) {
        if (subject.isInGroup("children")) {
            return polkit.Result.NO;
        } else {
            return polkit.Result.AUTH_SELF_KEEP;
        }
    }
});
```

运行一个外部助手来确定当前用户是否可能重启系统:
```js
polkit.addRule(function(action, subject) {
    if (action.id.indexOf("org.freedesktop.login1.reboot") == 0) {
        try {
            // user-may-reboot exits with success (exit code 0)
            // only if the passed username is authorized
            polkit.spawn(["/opt/company/bin/user-may-reboot",
                          subject.user]);
            return polkit.Result.YES;
        } catch (error) {
            // Nope, but do allow admin authentication
            return polkit.Result.AUTH_ADMIN;
        }
    }
});
```

下面的例子展示了授权决策如何依赖于pkexec(1)机制传递的变量:
```js
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.policykit.exec" &&
        action.lookup("program") == "/usr/bin/cat") {
        return polkit.Result.AUTH_ADMIN;
    }
});
```

下面的示例展示了从该机制传递的变量的另一种用法。在这种情况下，机制是UDisks，它定义了一组用于匹配的操作和变量:
```js
// Allow users in group 'engineers' to perform any operation on
// some drives without having to authenticate
//
polkit.addRule(function(action, subject) {
    if (action.id.indexOf("org.freedesktop.udisks2.") == 0 &&
        action.lookup("drive.vendor") == "SEAGATE" &&
        action.lookup("drive.model") == "ST3300657SS" &&
        subject.isInGroup("engineers")) {
            return polkit.Result.YES;
        }
    }
});
```


