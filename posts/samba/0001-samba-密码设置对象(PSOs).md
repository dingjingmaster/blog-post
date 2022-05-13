---
title: "Samba 密码设置对象(PSOs)"
date: 2022-05-13T09:34:29+08:00
tags: ['samba', 'PSOs', 'PSO']
categories: ['samba']
draft: false
---

## Samab密码设置对象
> Samab密码设置对象(Password Settings Objects，简称PSOs)，开始于 `Samba Version:4.9`

密码设置对象(PSO)是一种 AD 功能，也称为细粒度密码策略(FGPP)。在 AD 中，密码设置控制以下几个方面：
- 用户更改密码时的最小密码长度和复杂性要求
- 密码历史时长：放置用户再次使用以前的密码
- 密码使用的最小和最大期限：用户必须更改密码的频率
- 账户锁定：在将用户锁定其账户之前登录尝试失败的阈值，以及他们被锁定的持续时间。

在支持 PSO 之前，Samba 管理员只能为域中的所有用户配置密码设置。例如。如果您想强制系统管理员拥有更长、更安全的密码，那么每个用户都必须遵守相同的密码要求。

PSO 允许 AD 管理员覆盖域的密码策略设置，并为特定用户或用户组配置更精细的密码设置。例如，PSO 可以强制某些用户使用更长的密码长度，或者放宽对其他用户的复杂性限制，等等。 PSO 可以应用于组或单个用户。

## 如何配置

PSO 可以使用 `samba-tool domain passwordsettings pso` 命令设置。参看 `samba-tool domain passwordsettings pso --help` 获取更多使用细节。PSO 命令的功能：
- 自行管理 PSO，即使用 `create` 或 `set` 子命令配置密码设置。还有 `delete`、`list` 和 `show` 命令。
- 控制适用于特定用户的 PSO。使用 `apply` 和 `unapply` 将 PSO 链接到特定组或用户

许多不同的 PSO 可以应用于同一用户（直接或通过组）。当多个 PSO 应用于同一用户时，本质上是具有最低优先级的 PSO 生效。但是，直接应用于用户的 PSO 总是胜过通过组成员身份继承的 PSO。要查看对给定用户生效的 PSO，使用 `samba-tool domain passwordsettings pso show-user`。

如果没有 PSO 应用于用户，则应用域密码设置。您可以使用`samba-tool domain passwordsettings show|set`查看/修改这些配置。
注意：
- 请使用 `samba-tool` 创建密码设置对象，而不是使用手动 `LDIF`。 PSO 需要驻留在“密码设置容器”中，`samba-tool` 将自动对其进行排序。任何在别处创建的 PSO 都将被忽略。
- 内置组不包括在 PSO 计算中。任何应用于内置组的 PSO 都不会生效。

要使用 Windows GUI 创建 PSO，请打开 Active Directory 管理中心，然后导航到系统 -> 密码设置容器，然后单击“新建”按钮将弹出一个向导。

## 已知问题

当前 (v4.9) 配置 PSO 并将其应用于用户会导致性能下降。这是我们希望在未来解决的问题。

计算 PSO 涉及计算用户的组成员资格，这是一个相当昂贵的计算。更糟糕的是，我们可能需要多次查找 msDS-ResultantPSO，因此它可能会尝试多次计算组，仅针对一次用户身份验证操作。注意：
- 如果根本没有 PSO 对象，则不会影响性能。
- 我们尝试缓存 msDS-ResultantPSO 结果，因此每次用户身份验证操作只计算一次。但是，如果没有 msDS-ResultantPSO（即域默认值适用于用户），我们不会缓存结果。所以这将是最坏的情况：域中存在一些 PSO，但它们对大多数用户不起作用。
- 如果 PSO 直接应用于用户（而不是组），则跳过昂贵的组计算。但是，与将 PSO 应用于组相比，将 PSO 直接应用于用户会使 PSO 更难管理。

如果密码历史长度从零变为非零值，Windows 和 Samba 行为之间还有一个非常微不足道的错误 (#13431)。

## 代码位置

主要 PSO 行为是使用构造属性控制的：msDS-ResultantPSO。这是为用户对象生成的，其值是适用于该用户的 PSO 的 DN。因此，确定适用于用户的正确 PSO 的逻辑存在于 operation.c 中，在那里生成构造的属性。

如果你 `grep 'msDS-ResultantPSO'` 的代码库，你应该找到所有尝试使用它的地方。基本上，无论代码检查密码设置属性（例如 lockOutObservationWindow），我们都必须检查 PSO 是否适用于该用户，如果是，则使用 PSO 属性（具有不同的名称，即 msDS-LockoutObservationWindow）。 `samdb_result_effective_badPwdCount()` 就是一个很好的例子。

`get_pso_data_callback()` 也值得注意，因为它用 PSO 的值覆盖了 `dsdb_user_pwd_settings` 结构的值（即域密码设置）。在密码哈希模块中使用此结构（作为结构 `dsdb_control_password_change_status` 的一部分）来检查密码更改操作。

## 参考文档

- MS-ADTS 记录了如何确定 msDS-ResultantPSO（第 3.1.1.4.5.36 节）。
- MS-SAMR 记录了 msDS-ResultantPSO 如何用于确定应用于用户的有效密码设置，即 Effective-LockoutThreshold、Effective-MinimumPasswordLength 等。这在第 3.1.1.5 节用于发起更新约束的密码设置属性中记录，但这些 Effective-XYZ 值在整个文档的其他几个地方都被引用，例如在计算 userAccountControl 时。

