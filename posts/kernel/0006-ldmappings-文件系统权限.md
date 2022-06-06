---
title: "ldmappings 文件系统权限"
date: 2022-05-30T17:48:05+08:00
tags: ['linux', 'kernel', 'filesystem', '文件系统']
categories: ['kernel']
draft: false
---

## ldmappings
大多数文件系统开发人员都遇到过`idmapping`。当从磁盘读取或写入权限、向用户空间报告所有权或用于权限检查时使用它们。本文的目标读者是想知道idmappings如何工作的文件系统开发人员。

### formal notes

`idmapping`本质上是将一个id范围转换为另一个或相同范围的id。在用户空间中广泛使用的idmappings表示法惯例是:
```shell
u:k:r
```
- u 表示上层idmapset u的第一个元素
- k 表示下层idmapset k的第一个元素
- r 参数表示idmapping的范围，即映射了多少个id。

从现在开始，我们总是在id前加上u或k，以明确我们谈论的是上idmapset中的id还是下idmapset中的id。

为了看看实际情况，让我们看看下面的idmapping:
```shell
u22:k10000:r3
```
并写下它将生成的映射:
```shell
u22 -> k10000
u23 -> k10001
u24 -> k10002
```

从数学的观点来看，`U` 和 `K` 是有序集，而idmapping是 `U` 到 `K` 的有序同构，所以 `U` 和 `K` 是有序同构的。事实上，`U` 和 `K` 总是一个给定系统中所有可用`id`集合的有序子集。

简单地从数学角度分析这个问题有助于我们突出一些属性，这些属性使我们更容易理解如何在`idmapping`之间进行转换。例如，我们知道逆`idmapping`也是一个有序同构:
```shell
k10000 -> u22
k10001 -> u23
k10002 -> u24
```

考虑到我们处理的是有序同构，加上我们处理的是子集，我们可以相互嵌入`idmapping`，也就是说，我们可以在不同的`idmapping`之间进行合理的转换。例如，假设我们已经得到了三个`idmappings`:
```shell
1. u0:k10000:r10000
2. u0:k20000:r10000
3. u0:k30000:r10000
```

以及第一次`idmapping`产生的`id k11000`，它将`上idmapset`中的`u1000`映射到`下idmapset`中的`k11000`。

因为我们处理的是有序同构子集，所以询问第二个或第三个`idmapping`中`id k11000`对应的是什么是有意义的。使用的简单算法是应用第一个`idmapping`的逆，将`k11000`映射到`u1000`。然后，我们可以使用第二个`idmapping`映射或第三个`idmapping`映射来映射`u1000`。第二个`idmapping`将`u1000`映射到`21000`。第三个`idmapping`将`u1000`映射到`u31000`。

如果我们给以下三个`idmappings`相同的任务:
```shell
1. u0:k10000:r10000
2. u0:k20000:r200
3. u0:k30000:r300
```
我们将无法进行转换，因为这些集合不再是第一个`idmapping`的整个范围内的有序同构(然而，它们是第二个`idmapping`的整个范围内的有序同构。)第二个或第三个`idmapping`在上层`idmapset u`中都不包含`u1000`。这相当于没有id映射。我们可以简单地说`u1000`在第二个和第三个`idmapping`中是未映射的。内核将向用户空间报告未映射的id为`overflowuid` (`uid_t`)`-1`或`overflowgid` (`gid_t`)`-1`。

计算给定id映射到什么位置的算法非常简单。首先，我们需要验证范围是否可以包含目标id。为了简单起见，我们将跳过这一步。之后，如果我们想知道id映射到什么，我们可以做一些简单的计算:
- 如果我们想要从左到右映射:
```c
u:k:r
id - u + k = n
```
- 如果我们想要从右到左映射:
```c
u:k:r
id - k + u = n
```

除了“从左到右”，我们还可以说“向下”，除了“从右到左”，我们还可以说“向上”。很明显，向下和向上的映射是相反的。

要查看上面的简单公式是否有效，请考虑以下两个idmappings:
```shell
1. u0:k20000:r10000
2. u500:k30000:r10000
```

假设我们在第一个`idmapping`的`下idmapset`中给定`k21000`。我们想知道这是从第一个`idmapping`的`上层idmapset`中的哪个`id`映射过来的。我们在第一个`idmapping`中向上映射:
```shell
id     - k      + u  = n
k21000 - k20000 + u0 = u1000
```

现在假设我们在第二个`idmapping`的`上层idmapset`中有一个`id u1100`，我们想知道这个id在第二个`idmapping`的`下层idmapset`中映射到什么。这意味着我们在第二个`idmapping`中向下映射:
```shell
id    - u    + k      = n
u1100 - u500 + k30000 = k30600
```

### General note

在内核环境中，idmapping可以被解释为将一系列用户空间id映射到一系列内核id:

```shell
userspace-id:kernel-id:range
```
用户空间`id`总是`uid_t`或`gid_t`类型的`idmap`的上层`idmapset`中的一个元素，而内核`id`总是`kuid_t`或`kgid_t`类型的`idmapset`的下层`idmapset`中的一个元素。从现在开始，“用户空间id”将用于表示众所周知的`uid_t`和`gid_t`类型，而“内核id”将用于表示`kuid_t`和`kgid_t`。

内核主要关心内核id。它们在执行权限检查时使用，并存储在`inode`的`i_uid`和`i_gid`字段中。另一方面，用户空间id是一个由内核报告给用户空间的id，或者由用户空间传递给内核的id，或者从磁盘写入或读取的原始设备id。

注意，我们只关心内核存储`idmappings`的方式，而不关心用户空间如何指定它们。

在本文档的其余部分中，我们将在所有`用户空间id`前加上`u`，在所有`内核id`前加上`k`。`idmappings`范围将以`r`为前缀。因此，`idmapping`将被写成`u0:k10000:r10000`。

例如，id `u1000`是`idmapset上层`或以`u1000`开头的“userspace idmapset”中的id。它被映射到`k11000`，这是一个`内核id`，位于较低的idmapset或“内核idmapset”中，以`k10000`开头。

`内核id`总是由一个`idmapping`创建的。这样的`id映射`与用户名称空间相关联。因为我们主要关心`idmappings`是如何工作的，所以我们不需要关心`idmappings`是如何创建的，也不需要关心在文件系统上下文之外如何使用它们。这最好留给用户名称空间来解释。

初始用户命名空间是特殊的。它总是有一个如下形式的idmapping:
```shell
u0:k0:r4294967295
```

它是在这个系统上所有可用id范围上的身份映射。

其他用户名称空间通常有非标识的id映射，例如:
```shell
u0:k10000:r10000
```

当进程创建或想要更改文件的所有权时，或者当文件系统从磁盘读取文件的所有权时，根据与相关用户名称空间相关联的idmapping，用户空间id立即被转换为内核id。

例如，考虑由文件系统存储在磁盘上的文件被u1000所拥有:
- 如果一个文件系统要挂载在初始用户名称空间中(就像大多数文件系统那样)，那么将使用初始`idmapping`。正如我们看到的，这是简单的身份映射。这意味着从磁盘读取的id `u1000`将被映射到id `k1000`。因此，`inode`的`i_uid`和`i_gid`字段将包含`k1000`。
- 如果要以`u0:k10000:r10000`的id映射挂载文件系统，那么从磁盘读取的`u1000`将被映射到`k11000`。所以一个`inode`的`i_uid`和`i_gid`将包含`k11000`。

### 转换算法
我们已经简要地看到，可以在不同的idmapping之间进行转换。现在我们将进一步了解它是如何工作的。

#### 交叉映射
内核在很多地方都使用这种转换算法。例如，当通过`stat()`系统调用系列向用户空间报告文件的所有权时，使用它。

如果我们从一个`idmapping`中得到`k11000`我们可以把这个id映射到另一个`idmapping`中。为了使它工作，两个`idmapping`需要在它们的内核`idmapset`中包含相同的内核id。例如，考虑以下idmappings:
```shell
1. u0:k10000:r10000
2. u20000:k10000:r10000
```

我们在第一个`idmapping`中将`u1000`映射到`k11000`。然后，我们可以使用第二个`idmapping`的内核`idmapset`将`k11000`转换为第二个`idmapping`中的用户空间id:
```shell
/* Map the kernel id up into a userspace id in the second idmapping. */
from_kuid(u20000:k10000:r10000, k11000) = u21000
```

注意，我们如何通过颠倒算法在第一个idmapping中返回内核id:
```shell
/* Map the userspace id down into a kernel id in the second idmapping. */
make_kuid(u20000:k10000:r10000, u21000) = k11000

/* Map the kernel id up into a userspace id in the first idmapping. */
from_kuid(u0:k10000:r10000, k11000) = u1000
```

这个算法允许我们回答这样一个问题:给定的内核id对应于给定的`idmapping`中的哪个用户空间id。为了能够回答这个问题，两个`idmapping`都需要在各自的内核`idmapset`中包含相同的内核id。

例如，当内核从磁盘读取一个原始用户空间id时，它会根据与文件系统相关联的idmapping将其映射到内核id。让我们假设文件系统的id映射为`u0:k20000:r10000`，它从磁盘读取`u1000`拥有的文件。这意味着`u1000`将映射到`k21000`, `k21000`将存储在`inode`的`i_uid`和`i_gid`字段中。

当用户空间中的某人调用`stat()`或相关函数来获取文件的所有权信息时，内核不能简单地根据文件系统的`idmapping`来映射id，因为如果调用者使用`idmapping`，这会给出错误的所有者。

因此，内核将把id映射回调用者的idmapping中。让我们假设调用者有一个稍微不寻常的idmapping `u3000:k20000:r10000`，那么`k21000`将映射回`u4000`。因此，用户会看到这个文件属于`u4000`。

#### 重新映射
通过两个`idmapping`的用户空间`idmapset`，可以将一个`内核id`从一个`idmapping`转换为另一个`idmapping`。这相当于`重新映射内核id`。

让我们来看一个例子。我们给出了以下两个`idmappings`:
```c
1. u0:k10000:r10000
2. u0:k20000:r10000
```
我们在第一个`idmapping`中得到`k11000`。为了将第一个`idmapping`中的`内核id`转换为第二个`idmapping`中的`内核id`，我们需要执行两个步骤:
1. 在第一个idmapping中将内核id映射到用户空间id:
```shell
/* Map the kernel id up into a userspace id in the first idmapping. */
from_kuid(u0:k10000:r10000, k11000) = u1000
```
2. 在第二个idmapping中将用户空间id映射到内核id:
```shell
/* Map the userspace id down into a kernel id in the second idmapping. */
make_kuid(u0:k20000:r10000, u1000) = k21000
```

如您所见，我们在两个idmapping中都使用了用户空间idmapset来将一个idmapping中的内核id转换为另一个idmapping中的内核id。

这允许我们回答这样一个问题:我们需要使用哪个内核id才能在另一个idmapping中获得相同的用户空间id。为了回答这个问题，两个idmapping都需要在各自的用户空间idmapset中包含相同的用户空间id。

注意，在第一个idmapping中，我们可以通过颠倒算法轻松地返回内核id:
1. 在第二个idmapping中将内核id映射到用户空间id:
```shell
/* Map the kernel id up into a userspace id in the second idmapping. */
from_kuid(u0:k20000:r10000, k21000) = u1000
```
2. 在第一个idmapping中将用户空间id映射到内核id:
```shell
/* Map the userspace id down into a kernel id in the first idmapping. */
make_kuid(u0:k10000:r10000, u1000) = k11000
```

观察这种转换的另一种方法是，如果两个idmapping都有相关的用户空间id映射，则将其视为一个idmapping的倒置和另一个idmapping的应用。在使用idmapped挂载时，这将会派上用场。

### 非法转换

在一个idmapping的内核idmapset中使用一个id作为另一个或相同idmapping的用户空间idmapset中的id永远是无效的。内核idmapset总是表示内核id空间中的一个idmapset，而用户空间idmapset表示用户空间id。所以下面的翻译是被禁止的:
```
/* Map the userspace id down into a kernel id in the first idmapping. */
make_kuid(u0:k10000:r10000, u1000) = k11000

/* INVALID: Map the kernel id down into a kernel id in the second idmapping. */
make_kuid(u10000:k20000:r10000, k110000) = k21000
                                ~~~~~~~
```
和同样是错误的:
```
/* Map the kernel id up into a userspace id in the first idmapping. */
from_kuid(u0:k10000:r10000, k11000) = u1000

/* INVALID: Map the userspace id up into a userspace id in the second idmapping. */
from_kuid(u20000:k0:r10000, u1000) = k21000
                            ~~~~~
```

## 创建文件系统对象时的Idmappings
id向下映射或向上映射的概念在文件系统开发人员非常熟悉的两个内核函数中表达，我们已经在本文档中使用了它们:
```
/* Map the userspace id down into a kernel id. */
make_kuid(idmapping, uid)

/* Map the kernel id up into a userspace id. */
from_kuid(idmapping, kuid)
```
我们将简要介绍idmappings如何创建文件系统对象。为了简单起见，我们将只研究当VFS在调用文件系统本身之前已经完成路径查找时发生的情况。因此，我们关心的是调用`vfs_mkdir()`时会发生什么。我们还将假设创建文件系统对象的目录对每个人都是可读可写的。

当创建一个文件系统对象时，`调用者`将查看调用者的`文件系统id`。这些只是普通的`uid_t`和`gid_t`用户空间id，但它们在确定文件所有权时被专门使用，这就是为什么它们被称为“*文件系统id*”。它们通常与调用者的`uid`和`gid`相同，但也可以不同。我们将只假设它们总是相同的，以避免迷失在太多的细节中。

当调用者进入内核时，会发生两件事:
1. 在调用者的idmapping中将调用者的用户空间id向下映射到内核id。(准确地说，内核只会查看隐藏在当前任务凭证中的内核id，但对于我们的教育，我们将假设这个转换是及时发生的。)
2. 验证调用者的内核id可以映射到文件系统idmapping中的用户空间id。

第二步很重要，因为常规文件系统在写入磁盘时最终需要将内核id映射回用户空间id。因此，在第二步中，内核保证可以将有效的用户空间id写入磁盘。如果不能，内核将拒绝创建请求，甚至不冒远程文件系统损坏的风险。

精明的读者应该已经意识到这只是我们在上一节中提到的交叉映射算法的一个变种。首先，内核根据调用者的idmapping将调用者的用户空间id映射到内核id，然后根据文件系统的idmapping将内核id映射到内核id。

### 例1
```
caller id:            u1000
caller idmapping:     u0:k0:r4294967295
filesystem idmapping: u0:k0:r4294967295
```
调用者和文件系统都使用标识idmapping:
1. 在调用者的idmapping中将调用者的用户空间id映射到内核id:
```
make_kuid(u0:k0:r4294967295, u1000) = k1000
```
2. 验证调用者的内核id可以映射到文件系统idmapping中的用户空间id。
对于第二步，内核将调用`fsuidgid_has_mapping()`函数，最终归结为调用`from_kuid()`:
```
from_kuid(u0:k0:r4294967295, k1000) = u1000
```
在本例中，两个idmappings是相同的，所以没有什么令人兴奋的事情发生。最终，放置在磁盘上的用户空间id将是u1000。

### 例2
```
caller id:            u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k20000:r10000
```
1. 在调用者的idmapping中将调用者的用户空间id映射到内核id:
```
make_kuid(u0:k10000:r10000, u1000) = k11000
```
2. 验证调用者的内核id可以映射到文件系统的idmapping中的用户空间id:
```
from_kuid(u0:k20000:r10000, k11000) = u-1
```
很明显，虽然调用者的用户空间id可以在调用者的idmapping中成功映射到内核id，但内核id不能根据文件系统的idmapping进行映射。因此，内核将拒绝这个创建请求。

请注意，虽然这个示例不太常见，但由于大多数文件系统不能使用非初始idmappings挂载，这是一个常见的问题，我们可以在下一个示例中看到。

### 例3
```
caller id:            u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k0:r4294967295
```
1. 在调用者的idmapping中将调用者的用户空间id映射到内核id:
```
make_kuid(u0:k10000:r10000, u1000) = k11000
```
2. 验证调用者的内核id可以映射到文件系统的idmapping中的用户空间id:
```
from_kuid(u0:k0:r4294967295, k11000) = u11000
```

我们可以看到，翻译总是成功的。文件系统最终放入磁盘的用户空间id将始终与调用者的idmapping中创建的内核id的值相同。这主要有两个后果。

首先，我们不能允许调用者最终使用另一个用户空间id写入磁盘。只有在使用调用者的或另一个idmapping挂载整个fileystem时才能这样做。但是该解决方案仅限于少数文件系统，而且不太灵活。但这是一个在容器化工作负载中非常重要的用例。

其次，调用者通常无法创建任何具有严格权限的文件或访问目录，因为在调用者的idmapping中，没有一个文件系统的内核id映射到有效的用户空间id

1. 在文件系统的idmapping中将原始用户空间id映射到内核id:
```
make_kuid(u0:k0:r4294967295, u1000) = k1000
```
2. 在调用者的idmapping中映射内核id到用户空间id:
```
from_kuid(u0:k10000:r10000, k1000) = u-1
```

### 例4
```
file id:              u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k0:r4294967295
```

为了向用户空间报告所有权，内核使用了上一节介绍的交叉映射算法:
1. 将磁盘上的用户空间id映射到文件系统idmapping中的内核id:
```
make_kuid(u0:k0:r4294967295, u1000) = k1000
```
2. 在调用者的idmapping中将内核id映射到用户空间id:
```
from_kuid(u0:k10000:r10000, k1000) = u-1
```

在这种情况下，交叉映射算法失败，因为文件系统idmapping中的内核id不能映射到调用者idmapping中的用户空间id。因此，内核将报告该文件的所有权为溢出。

### 例5
```
file id:              u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k20000:r10000
```

为了向用户空间报告所有权，内核使用了上一节介绍的交叉映射算法:
1. 将磁盘上的用户空间id映射到文件系统idmapping中的内核id:
```
make_kuid(u0:k20000:r10000, u1000) = k21000
```
2. 在调用者的idmapping中将内核id映射到用户空间id:
```
from_kuid(u0:k10000:r10000, k21000) = u-1
```

在这种情况下，交叉映射算法同样失败，因为文件系统idmapping中的内核id不能映射到调用者idmapping中的用户空间id。因此，内核将报告该文件的所有权为溢出。

注意，如果调用者使用初始idmapping，那么在最后两个示例中，事情将变得多么简单。对于使用初始idmapping安装的文件系统来说，这很简单。所以我们只考虑一个id映射为u0:k20000:r10000的文件系统:
1. 将磁盘上的用户空间id映射到文件系统idmapping中的内核id:
```
make_kuid(u0:k20000:r10000, u1000) = k21000
```
2. 在调用者的idmapping中将内核id映射到用户空间id:
```
from_kuid(u0:k0:r4294967295, k21000) = u21000
```

## idmapped挂载上的Idmappings
在上一节中我们看到的调用者的idmapping和文件系统的idmapping不兼容的例子会导致工作负载的各种问题。对于一个更复杂但常见的示例，考虑在主机上启动两个容器。为了完全防止这两个容器相互影响，管理员通常可以为这两个容器使用不同的不重叠的idmapping:
```
container1 idmapping:  u0:k10000:r10000
container2 idmapping:  u0:k20000:r10000
filesystem idmapping:  u0:k30000:r10000
```

管理员希望对以下文件集提供简单的读写访问:

```
dir id:       u0
dir/file1 id: u1000
dir/file2 id: u2000
```

到两个容器目前都不能。

当然，管理员可以选择通过chown()递归地更改所有权。例如，他们可以改变所有权，以便dir和它下面的所有文件可以从文件系统的交叉映射到容器的idmapping。让我们假设它们改变了所有权，以便与第一个容器的idmapping兼容:

```
dir id:       u10000
dir/file1 id: u11000
dir/file2 id: u12000
```
这仍然会使dir对第二个容器毫无用处。事实上，dir和它下面的所有文件将继续显示为第二个容器的溢出所有。

再来看看另一个越来越受欢迎的例子。一些服务管理器，比如systemd，实现了一个叫做“可移植主目录”的概念。用户可能希望在分配了不同登录用户空间id的不同机器上使用自己的主目录。大多数用户在家里的机器上将u1000作为登录id，并且他们主目录中的所有文件通常都属于u1000。在大学或工作单位，他们可能有另一个登录id，如u1125。这使得在他们的工作机器上与他们的主目录交互变得相当困难。

在这两种情况下，递归地改变所有权都有严重的影响。最明显的一个是所有权是全球性和永久性的变化。在主目录的情况下，所有权甚至需要在每次用户从他们的主目录切换到他们的工作机器时发生这种变化。对于非常大的文件集，这将变得越来越昂贵。

如果用户幸运的话，他们处理的文件系统是在用户名称空间内安装的。但是这也会全局地改变所有权，所有权的改变与文件系统挂载的生命周期有关，也就是超级块。更改所有权的惟一方法是完全卸载文件系统，然后在另一个用户名称空间中再次挂载它。这通常是不可能的，因为这意味着当前访问文件系统的所有用户都不能再访问了。这意味着dir仍然不能在具有不同idmapping的两个容器之间共享。但通常用户甚至没有这个选项，因为大多数文件系统在容器内是不可安装的。并且不要安装它们可能是可取的，因为它不需要文件系统处理恶意的文件系统映像。

但是上面提到的用例以及更多的情况都可以通过idmapped挂载来处理。它们允许在不同的坐骑上暴露同一套拥有不同所有权的dentry。这是通过通过mount_setattr()系统调用用用户名称空间标记挂载来实现的。然后使用与它相关联的idmapping从调用者的idmapping转换到文件系统的idmapping，然后使用我们前面介绍的重新映射算法进行反向转换。

Idmapped挂载使得以一种临时和本地化的方式改变所有权成为可能。所有权的变更仅限于一个特定的坐骑，并且与坐骑的生命周期相关。暴露文件系统的所有其他用户和位置都不受影响。

支持idmapped挂载的文件系统没有任何真正的理由来支持在用户名称空间内被挂载。可以在idmapped挂载下完全公开文件系统，以获得相同的效果。这样做的好处是，文件系统可以将超级块的创建留给初始用户名称空间中的特权用户。

但是，完全可以将idmapped挂载与用户名称空间内可挂载的文件系统结合起来。我们将在下面进一步讨论这个问题。

### 重新映射 helpers
添加了Idmapping函数，在Idmapping之间进行转换。它们使用了我们前面介绍过的重新映射算法。我们来看看两个例子:
- `i_uid_into_mnt()` 和 `i_gid_into_mnt()`
`i_*id_into_mnt()`函数将文件系统的内核id转换为挂载的idmapping中的内核id:
```
/* Map the filesystem's kernel id up into a userspace id in the filesystem's idmapping. */
from_kuid(filesystem, kid) = uid

/* Map the filesystem's userspace id down ito a kernel id in the mount's idmapping. */
make_kuid(mount, uid) = kuid
```
- `mapped_fsuid()` 和 `mapped_fsgid()`
`mapped_fs*id()`函数将调用者的内核id转换为文件系统idmapping中的内核id。这个转换是通过使用挂载的idmapping重新映射调用者的内核id来实现的:
```
/* Map the caller's kernel id up into a userspace id in the mount's idmapping. */
from_kuid(mount, kid) = uid

/* Map the mount's userspace id down into a kernel id in the filesystem's idmapping. */
make_kuid(filesystem, uid) = kuid
```
注意，这两个函数是相反的。考虑以下idmappings:
```
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k20000:r10000
mount idmapping:      u0:k10000:r10000
```

假设从磁盘读取属于`u1000`的文件。文件系统根据它的`idmapping`将这个id映射到`k21000`。这是存储在`inode`的`i_uid`和`i_gid`字段中的内容。

当调用者通过`stat()`查询这个文件的所有权时，内核通常会简单地使用交叉映射算法，并将文件系统的内核id映射到调用者的idmapping中的用户空间id。

但是当调用者访问idmapped挂载上的文件时，内核会首先调用`i_uid_into_mnt()`，从而将文件系统的内核id转换成挂载的idmapping中的内核id:
```
i_uid_into_mnt(k21000):
  /* Map the filesystem's kernel id up into a userspace id. */
  from_kuid(u0:k20000:r10000, k21000) = u1000

  /* Map the filesystem's userspace id down ito a kernel id in the mount's idmapping. */
  make_kuid(u0:k10000:r10000, u1000) = k11000
```

最后，当内核向调用者报告所有者时，它将把挂载的idmapping中的内核id转换为调用者idmapping中的用户空间id:
```
from_kuid(u0:k10000:r10000, k11000) = u1000
```

我们可以通过验证在创建新文件时发生了什么来测试这个算法是否真的有效。假设用户正在创建一个u1000的文件。

内核将其映射到调用者的idmapping中的k11000。通常，内核现在会应用交叉映射，验证k11000可以映射到文件系统idmapping中的用户空间id。由于k11000不能直接映射到文件系统的idmapping中，所以创建请求失败。

但是当调用者访问idmapped挂载上的文件时，内核会首先调用`mapped_fs*id()`，从而根据挂载的idmapping将调用者的内核id转换成一个内核id:
```
mapped_fsuid(k11000):
   /* Map the caller's kernel id up into a userspace id in the mount's idmapping. */
   from_kuid(u0:k10000:r10000, k11000) = u1000

   /* Map the mount's userspace id down into a kernel id in the filesystem's idmapping. */
   make_kuid(u0:k20000:r10000, u1000) = k21000
```

当最后写入磁盘时，内核会将k21000映射到文件系统的idmapping中的用户空间id:
```
from_kuid(u0:k20000:r10000, k21000) = u1000
```

正如我们所看到的，我们最终得到了一个可逆的，因此信息保持的算法。在idmapped挂载上从u1000创建的文件也会被报告为u1000所拥有，反之亦然。

现在，让我们在idmapped挂载上下文中简要地重新考虑前面失败的例子。

### 例2 reconsider
```
caller id:            u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k20000:r10000
mount idmapping:      u0:k10000:r10000
```

当调用者使用非初始idmapping时，通常的情况是将相同的idmapping附加到挂载上。现在我们执行三个步骤:
1. 在调用者的idmapping中将调用者的用户空间id映射到内核id:
```
make_kuid(u0:k10000:r10000, u1000) = k11000
```
2. 将调用者的内核id转换为文件系统idmapping中的内核id:
```
mapped_fsuid(k11000):
  /* Map the kernel id up into a userspace id in the mount's idmapping. */
  from_kuid(u0:k10000:r10000, k11000) = u1000

  /* Map the userspace id down into a kernel id in the filesystem's idmapping. */
  make_kuid(u0:k20000:r10000, u1000) = k21000
```
3. 验证调用者的内核id可以映射到文件系统的idmapping中的用户空间id:
```
from_kuid(u0:k20000:r10000, k21000) = u1000
```

所以磁盘上的所有权是u1000。

### 例3 reconsidered
```
caller id:            u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k0:r4294967295
mount idmapping:      u0:k10000:r10000
```
同样的转换算法也适用于第三个例子。
1. 在调用者的idmapping中将调用者的用户空间id映射到内核id:
```
make_kuid(u0:k10000:r10000, u1000) = k11000
```
2. 将调用者的内核id转换为文件系统idmapping中的内核id:
```
mapped_fsuid(k11000):
   /* Map the kernel id up into a userspace id in the mount's idmapping. */
   from_kuid(u0:k10000:r10000, k11000) = u1000

   /* Map the userspace id down into a kernel id in the filesystem's idmapping. */
   make_kuid(u0:k0:r4294967295, u1000) = k1000
```
3. 验证调用者的内核id可以映射到文件系统的idmapping中的用户空间id:
```
from_kuid(u0:k0:r4294967295, k21000) = u1000
```
所以磁盘上的所有权是u1000。

### 例4 reconsidered
```
file id:              u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k0:r4294967295
mount idmapping:      u0:k10000:r10000
```
为了向用户空间报告所有权，内核现在使用我们前面介绍的转换算法执行三个步骤:
1. 将磁盘上的用户空间id映射到文件系统idmapping中的内核id:
```
make_kuid(u0:k0:r4294967295, u1000) = k1000
```
2. 将内核id转换为挂载的idmapping中的内核id:
```
i_uid_into_mnt(k1000):
  /* Map the kernel id up into a userspace id in the filesystem's idmapping. */
  from_kuid(u0:k0:r4294967295, k1000) = u1000

  /* Map the userspace id down into a kernel id in the mounts's idmapping. */
  make_kuid(u0:k10000:r10000, u1000) = k11000
```
3. 在调用者的idmapping中将内核id映射到用户空间id:
```
from_kuid(u0:k10000:r10000, k11000) = u1000
```

之前，调用者的内核id不能在文件系统的idmapping中交叉映射。有了idmapped挂载之后，现在可以通过挂载的idmapping将它交叉映射到文件系统的idmapping中。现在将根据挂载的idmapping使用u1000创建文件。

### 例5 reconsidered
```
file id:              u1000
caller idmapping:     u0:k10000:r10000
filesystem idmapping: u0:k20000:r10000
mount idmapping:      u0:k10000:r10000
```

同样，为了向用户空间报告所有权，内核现在使用我们前面介绍的转换算法执行三个步骤:
1. 将磁盘上的用户空间id映射到文件系统idmapping中的内核id:
```
make_kuid(u0:k20000:r10000, u1000) = k21000
```
2. 将内核id转换为挂载的idmapping中的内核id:
```
i_uid_into_mnt(k21000):
  /* Map the kernel id up into a userspace id in the filesystem's idmapping. */
  from_kuid(u0:k20000:r10000, k21000) = u1000

  /* Map the userspace id down into a kernel id in the mounts's idmapping. */
  make_kuid(u0:k10000:r10000, u1000) = k11000
```
3. 在调用者的idmapping中将内核id映射到用户空间id:
```
from_kuid(u0:k10000:r10000, k11000) = u1000
```
以前，文件的内核id不能在文件系统的idmapping中交叉映射。有了idmapped挂载之后，现在可以通过挂载的idmapping将它交叉映射到文件系统的idmapping中。根据挂载的idmapping，该文件现在属于u1000。

## 更改主目录的所有权

我们在上面已经看到了当调用者、文件系统或两者都使用非初始idmapping时，如何使用idmapped挂载在idmapping之间进行转换。当调用者使用非初始idmapping时，存在各种各样的用例。这通常发生在容器化工作负载的上下文中。结果就像我们看到的那样，对于使用初始idmapping挂载的文件系统和使用非初始idmapping挂载的文件系统，对文件系统的访问无法工作，因为内核id不能在调用者的和文件系统的idmapping之间交叉映射。

正如我们在上面看到的，idmapped挂载提供了一种解决方案，它根据挂载的idmapping重新映射调用者或文件系统的idmapping。

除了容器化的工作负载之外，idmapped挂载还有一个优点:当调用者和文件系统都使用初始idmapping时，它们也可以工作，这意味着主机上的用户可以在每次挂载的基础上改变目录和文件的所有权。

考虑我们前面的示例，其中用户的主目录位于可移植存储上。在家里，他们的id是u1000，在他们的主目录中的所有文件都属于u1000，而在uni或work，他们的登录id是u1125。

带着他们的主目录会有问题。它们不能轻松地访问它们的文件，如果不应用宽松的权限或acl，它们可能无法写入磁盘，而且即使它们可以这样做，它们也将以u1000和u1125拥有的文件和目录混合而结束。

Idmapped挂载允许解决这个问题。用户可以在他们的工作计算机或家里的计算机上为他们的主目录创建idmapped挂载，这取决于他们希望最终在便携存储本身上拥有什么所有权。

假设他们希望磁盘上的所有文件都属于u1000。当用户在他们的工作岗位插入便携存储时，他们可以设置一个作业，该作业创建一个idmapped挂载，其中的idmapping最小值为u1000:k1125:r1。所以现在当他们创建一个文件时，内核执行以下步骤，我们已经从上面知道::
```
caller id:            u1125
caller idmapping:     u0:k0:r4294967295
filesystem idmapping: u0:k0:r4294967295
mount idmapping:      u1000:k1125:r1
```
1. 在调用者的idmapping中将调用者的用户空间id映射到内核id:
```
make_kuid(u0:k0:r4294967295, u1125) = k1125
```
2. 将调用者的内核id转换为文件系统idmapping中的内核id:
```
mapped_fsuid(k1125):
  /* Map the kernel id up into a userspace id in the mount's idmapping. */
  from_kuid(u1000:k1125:r1, k1125) = u1000

  /* Map the userspace id down into a kernel id in the filesystem's idmapping. */
  make_kuid(u0:k0:r4294967295, u1000) = k1000
```
3. 验证调用者的内核id可以映射到文件系统的idmapping中的用户空间id:
```
from_kuid(u0:k0:r4294967295, k1000) = u1000
```

因此，最终将在磁盘上创建u1000文件。

现在让我们简单地看看id为u1125的调用者将在他们的工作计算机上看到什么所有权:
```
file id:              u1000
caller idmapping:     u0:k0:r4294967295
filesystem idmapping: u0:k0:r4294967295
mount idmapping:      u1000:k1125:r1
```
1. 将磁盘上的用户空间id映射到文件系统idmapping中的内核id:
```
make_kuid(u0:k0:r4294967295, u1000) = k1000
```
2. 将内核id转换为挂载的idmapping中的内核id:
```
i_uid_into_mnt(k1000):
  /* Map the kernel id up into a userspace id in the filesystem's idmapping. */
  from_kuid(u0:k0:r4294967295, k1000) = u1000

  /* Map the userspace id down into a kernel id in the mounts's idmapping. */
  make_kuid(u1000:k1125:r1, u1000) = k1125
```
3. 在调用者的idmapping中将内核id映射到用户空间id:
```
from_kuid(u0:k0:r4294967295, k1125) = u1125
```
因此，最终将报告调用者文件属于u1125，在我们的示例中，u1125是调用者工作站上的用户空间id。

放置在磁盘上的原始用户空间id是u1000，因此当用户将他们的主目录返回到他们的主计算机时，他们使用初始idmapping分配了u1000，并使用初始idmapping挂载文件系统，他们将看到u1000拥有的所有文件。
