---
title: "Polkit 介绍"
date: 2022-06-07T18:50:53+08:00
tags: ['linux', 'polkit']
categories: ['linux']
draft: false
---
## polkit是什么?

polkit提供了一个授权API，旨在由特权程序(“机制”)使用，为非特权程序(“客户端”)提供服务。查看polkit手册页面了解系统架构和总体情况。

### polkit 程序

Polkit 应用程序是使用 Polkit 权限作为决策组件的应用程序(polkitd)。他们通过将`.policy`文件安装到`/usr/share/polkit-1/actions`目录中，并在运行时与`polkitd`进行通信(通过D-Bus API或间接通过libpolkit-gobject-1库或pkcheck命令)。

### polkit 适用场景

- 如果您正在编写特权机制(即以root身份运行或具有特殊权限)，打算由非特权程序使用，则使用polkit。
- 认真考虑定义什么行为。在许多情况下，操作和polkit动作之间没有1:1的映射。通常，polkit操作与操作所处理的对象的关系比操作本身的关系更大。在粒度过细和粒度过粗之间取得适当的平衡是很重要的。
- 尝试选择操作和隐式授权，以便使用您的机制的应用程序能够为在控制台登录的用户开箱即用。应该优先考虑不使用身份验证对话框打断控制台用户。例如，对于添加打印机队列这样的普通任务，要求控制台用户进行身份验证是不明智的(如果管理员真的希望操作系统以这种方式工作，他总是可以部署合适的授权规则)。
- 考虑所选择的隐式授权对多用户系统的影响。一般情况下，普通用户既不能为其他用户修改重要系统的行为/配置，也不能查看其他用户的私人数据。如果您的应用程序需要授权框架，那么至少在某些情况下，默认配置很可能会拒绝授权。默认使用`auth_admin` 而不是`auth_self`。(在单用户桌面中，单个用户通常被配置为polkit管理员，因此这两种变体的行为是相同的。在多用户系统中，非管理员用户将受到默认配置的限制。)
- 将polkit变量与`CheckAuthorization()`请求一起传递，这样就可以编写与这些请求匹配的授权规则。还要在文档中记录这些变量(例如，请参阅udisks2操作和变量)。
- 传递一个定制的身份验证消息(使用`polkit.message` 和 ` polkit.gettext_domain` 变量)，它包含比`.policy`文件的`message`元素中声明的请求更详细的信息。例如：显示 “需要身份验证来格式化INTEL SSDSA2MH080G1GC (/dev/sda)”，而不是仅仅显示“需要身份验证来格式化设备”。
- 确保您的应用程序工作，即使当`org.freedesktop.PolicyKit1 D-Bus` 服务不可用(这可能发生，如`polkitd(8)`没有安装或`polkit.service`被屏蔽)。如果你使用libpolkit-gobject-1库，这意味着处理`polkit_authority_get_sync()`或`polkit_authority_get_finish(`)返回`NULL`或`polkit_authority_check_authorization()` / `polkit_authority_check_authorization_sync()`失败，返回错误不在`POLKIT_ERROR`域。处理polkit权限不可用的一种适当方法可以是只允许uid 0执行操作，禁止所有操作或其他操作。

### polkit 不适用的场景

- 如果您的程序不打算供非特权程序使用，则不要使用polkit。例如，如果您正在编写开发人员工具或低级核心OS命令行工具，那么只要求用户是root就可以了。用户可以通过sudo(8)， pkexec(1)或编写一个简单的使用polkit的机制来访问工具的(安全)子集。
- 除非必要，否则不要使用polkit。换句话说，并不是每个为非特权程序提供服务的特权程序都必须使用polkit。
- 不要在每次授权机构发出`change`信号时对所有操作调用`CheckAuthorization()`。这不仅是对资源的浪费，结果也可能不准确，因为授权规则可以在任何时候返回它们想要的任何内容。
- 在等待权限回复时，不要阻塞你的主线程(例如用于服务来自非特权程序的IPC请求的主线程)——`CheckAuthorization()`调用可能需要很长时间(秒，甚至分钟)才能完成，因为可能涉及用户交互。相反，可以使用异步API，或者使用带有同步API的专用线程。
- 不要在应用程序中包含任何授权规则，因为这只适用于管理员和特殊用途的操作系统/环境。有关更多信息，请参阅“授权规则”一节。

### 在非特权程序中的使用

非特权程序通常不直接使用polkit——它只是调用特权机制，该机制在使用polkit进行检查(可能包括显示身份验证对话框)后呈现服务(或拒绝请求)。在这种设置中，非特权程序不知道正在使用polkit—它只是等待特权机制执行请求(如果涉及身份验证对话框，这可能需要许多秒)。这是一件好事，因为不必担心诸如polkit之类的实现细节，这有助于简化非特权程序。

有时，没有特权的程序需要禁用、修改或删除UI元素，以向用户传达某些操作无法执行(例如，用户没有获得授权)或需要验证(例如，在UI中显示挂锁图标)。在这种情况下，最好的方法通常是让非特权程序从特权机制(而不是polkit)获得该信息。这一点尤其正确，因为通常没有可靠的方法可以让非特权程序知道将要使用什么polkit操作。一般来说,不能保证操作(如d-bus方法)映射 `1:1:` 到 polkit 的某个 action ——例如,一个磁盘管理器服务的`Format()`方法可能检查`net.company.diskmanager.format-removable` 磁盘是否可移除 和 `net.company.diskmanager.format-fixed` 磁盘是固定的格式。

然而，在某些情况下，例如在使用`org.freedesktop.policykit.imply` (参见polkit(8)手册页)时，对于非特权程序查询polkit权限它是有意义的(如更新UI元素),只要无特权的程序不通过任何变量随着`CheckAuthorization()`调用，这种查询操作是被允许的(否则很容易欺骗身份验证对话框和旁路授权规则)。事实上，由于这个用例是如此常见，`libpolkit-gobject-1`提供了可以与`GtkLockButton`一起使用的`PolkitPermission`类型(它派生自GPermission)。注意，要使GtkLockButton正常工作，支持它的polkit操作应该使用`auth_admin_keep`作为隐式授权(或者很少使用`auth_self_keep`作为不影响其他用户的服务)。这通常用于实现一个即时应用范例，用户解锁(通过身份验证)，例如一个首选项窗格窗口，然后可以自由更改设置，直到授权过期或被撤销。

### 没有身份验证代理

如果一个polkit应用程序想要处理没有身份验证代理存在的情况(例如，如果应用程序是通过ssh(1)登录启动的)，应用程序可以使用PolkitAgentTextListener类型来根据需要生成自己的身份验证代理。另外，也可以使用pkttyagent(1) helper来完成此任务。

## 编写polkit认证代理

认证代理由桌面环境提供。当用户会话开始时，代理使用`RegisterAuthenticationAgent()`方法向`polkit Authority`注册。当需要服务时，机构将调用`org.freedesktop.PolicyKit1.AuthenticationAgent` D-Bus接口上的方法。一旦用户通过身份验证，(特权部分)代理将调用`AuthenticationAgentResponse2()`方法。这个方法应该被视为内部实现细节，调用者应该使用`PolkitAgentSession` API来调用它，它目前使用一个setuid助手程序。

libpolkit-agent-1库提供了帮助程序，使构建使用本机身份验证系统的身份验证代理变得容易，例如pam(8)。

如果设置了环境变量`POLKIT_DEBUG`，则`libpolkit-agent-1`库将在标准输出中输出诊断信息。









