---
title: "C++调用python"
date: 2022-08-06T08:52:54+08:00
tags: ['c++', 'python', 'python c-api']
categories: ['c&c++']
draft: true
---

## 说明

Python的应用程序编程接口为C和C++编程人员提供了在不同级别上访问Python解析器的权限。该API在C++中同样可用，它通常被统称为 Python/C API。
<br/>
使用 Python/C API有两个根本不同的原因：
1. 为了特定目的编写扩展 Python 解析器的C模块
2. 在更大的应用程序中使用 Python 作为组件，为C/C++引用中嵌入Python提供 API

> 注意: 文档内容来自 `Python 3.10.6`

### 代码标准

如果您正在编写包含在CPython中的C代码，则必须遵循PEP 7中定义的指导方针和标准。这些指导原则适用于任何版本的Python。您自己的第三方扩展模块不需要遵循这些约定，除非您最终希望将它们贡献给Python。

### 包含文件

使用Python/C API所需的所有函数、类型和宏定义都包含在你的代码中:

```c
#define PY_SSIZE_T_CLEAN
#include <Python.h>
```
这意味着包含了以下头文件`<stdio.h>`, `<string.h>`, `<errno.h>`, `<limits.h>`, `<assert.h>` 和 `<stdlib.h>`

> 注意:
> 由于Python可能会定义一些会在某些系统上影响标准头文件的预处理器定义，所以必须在包含任何标准头文件之前包含Python.h。
> 建议始终在包含`Python.h`之前定义`PY_SSIZE_T_CLEAN`。有关此宏的说明，请参阅解析参数和构建值。

由`Python.h`定义的所有用户可见名称(由包含的标准头文件定义的除外)都有一个前缀`Py`或`_Py`。以`_Py`开头的名称供Python实现内部使用，扩展编写人员不应使用。结构成员名没有保留前缀。

<br/>

头文件通常与`Python`一起安装。在Unix上，它们位于`prefix/include/pythonversion/`和`exec_prefix/include/pythonversion/`目录中，其中`prefix`和`exec_prefix`由`Python`的`configure`脚本的相应参数定义，`version`为`'%d.%d' % sys.version_info[2]`。在Windows上，头文件安装在`prefix/include`中，其中`prefix`是指定给安装程序的安装目录。

<br/>
要包含头文件，请将两个目录(如果不同)放在编译器的`include`搜索路径上。不要将父目录放在搜索路径上，然后使用`#include`;这将在多平台构建中中断，因为`prefix`下平台独立的头文件包含来自`exec_prefix`的平台特定头文件。

<br/>
c++用户应该注意，API完全是用C定义的，头文件已经将入口点声明为`extern "C"`。因此，使用c++中的API不需要做任何特殊的事情。

### 有用的宏

Python头文件中定义了几个有用的宏。许多函数的定义更接近它们的用处(例如`Py_RETURN_NONE`)。这里只包含部分。

#### `Py_UNREACHABLE()`

当您的代码路径无法通过设计到达时，可以使用此方法。例如，在switch语句中的default:子句中，所有可能的值都包含在case语句中。在您可能想要调用assert(0)或abort()的地方使用它。

<br/>

在发布模式下，宏帮助编译器优化代码，并避免关于不可访问代码的警告。例如，宏在发布模式下在GCC上使用`__builtin_unreachable()`实现。

<br/>

`Py_UNREACHABLE()`的使用是在调用一个永远不会返回但没有声明的函数`_Py_NO_RETURN`之后。

<br/>

如果代码路径不太可能是代码，但在特殊情况下可以到达，则一定不要使用此宏。例如，在内存不足的情况下，或者系统调用返回的值超出了预期范围。在这种情况下，最好向调用者报告错误。如果错误不能报告给调用者，则可以使用`Py_FatalError()`。

> 3.7版本新功能

#### `Py_ABS(x)`

返回 X 的绝对值

> 3.3 版本

#### `Py_MIN(x, y)`

> 3.3 版本

#### `Py_MAX(x, y)`

> 3.3 版本

#### `Py_STRINGIFY(x)`

将 x 转为 c 字符串

> 3.4 版本

#### `Py_MEMBER_SIZE(type, member)`

返回结构体`type`成员`member`字节大小

> 3.6 版本

#### `Py_CHARMASK(c)`

参数必须是[-128, 127] 或 [0, 255] 范围内的字符或整数。这个宏返回 `unsigned char` 类型。

#### `Py_GETENV(s)`

#### `Py_UNUSED(arg)`

#### `Py_DEPRECATED(version)`

例子:
```c
Py_DEPRECATED(3.8) PyAPI_FUNC(int) Py_OldFunction(void);
```
> 3.8 版本

#### `PyDoc_STRVAR(name, str)`

创建一个具有可在文档字符串中使用的名称的变量。如果Python构建时没有使用文档字符串，则该值将为空。

<br/>

使用`PyDoc_STRVAR`作为文档字符串来支持构建没有文档字符串的Python，如PEP 7中指定的那样。

例子:
```c
PyDoc_STRVAR(pop_doc, "Remove and return the rightmost element.");

static PyMethodDef deque_methods[] = {
    // ...
    {"pop", (PyCFunction)deque_pop, METH_NOARGS, pop_doc},
    // ...
}
```

#### `PyDoc_STR(str)`

为给定的输入字符串创建一个文档字符串，如果文档字符串被禁用，则创建一个空字符串。

<br/>

在指定文档字符串时使用`PyDoc_STR`来支持构建没有文档字符串的Python，如PEP 7中所指定的那样。

<br/>
例子:
```c
static PyMethodDef pysqlite_row_methods[] = {
    {"keys", (PyCFunction)pysqlite_row_keys, METH_NOARGS,
        PyDoc_STR("Returns the keys of the row.")},
    {NULL, NULL}
};
```

### 对象、类型、引用计数

大多数`Python/C` API函数都有一个或多个参数以及一个`PyObject*`类型的返回值。该类型是指向表示任意Python对象的不透明数据类型的指针。由于所有Python对象类型在大多数情况下都被Python语言以相同的方式处理(例如，赋值、作用域规则和参数传递)，所以它们应该由单一的C类型表示才是合适的。几乎所有Python对象都存在于*堆*中:你不能声明PyObject类型的`auto`或`static`变量，只有`PyObject*`类型的指针变量可以声明。唯一的例外是类型对象;由于这些对象永远不能被释放，它们通常是静态`PyTypeObject`对象。
<br/>

所有Python对象(甚至是Python整数)都有一个类型和一个引用计数。对象的类型决定了对象的类型(例如，整数、列表或用户定义的函数;在标准类型层次结构中解释了更多)。对于每个已知的类型，都有一个宏来检查对象是否属于该类型;例如，当(且仅当)a指向的对象是一个Python列表时，`PyList_Check(a)`为真。

#### 引用计数

引用计数很重要，因为当今计算机的内存大小有限(通常是非常有限的);它计算有多少不同的地方有一个对象的引用。这样的位置可以是另一个对象，或全局(或静态)C变量，或某个C函数中的局部变量。当一个对象的引用计数为零时，该对象将被释放。如果它包含对其他对象的引用，则它们的引用计数将减少。如果这些其他对象的引用计数减少为零，则可以依次释放这些对象。(这里有一个对象之间相互引用的明显问题;目前，解决方案是“不要那样做”。)
<br/>

引用计数总是显式地操作。通常的方法是使用宏`Py_INCREF()`将对象的引用计数`增加1`，使用`Py_DECREF()`将其`减少1`。

> `Py_DECREF()`宏比`incref`宏要复杂得多，因为它必须检查引用计数是否变为零，然后确定是否调用对象的释放器。释放器是包含在对象类型结构中的函数指针。特定于类型的释放器负责减少对象中包含的其他对象的引用计数(如果这是一个复合对象类型，比如列表)，以及执行任何需要的附加结束。引用计数不会溢出;至少使用与虚拟内存中不同内存位置相同的位来保存引用计数(假设`sizeof(Py_ssize_t) >= sizeof(void*)`)。因此，增加引用计数是一个简单的操作。

对于每个包含指向对象指针的局部变量，不需要递增对象的引用计数。理论上，当变量指向对象时，对象的引用计数增加1，而当变量超出范围时，对象的引用计数减少1。但是，这两者相互抵消，所以最后引用计数没有改变。使用引用计数的唯一真正原因是，只要变量指向对象，就防止对象被释放。如果我们知道该对象至少有一个其他引用的寿命至少与我们的变量一样长，那么就不需要临时增加引用计数。出现这种情况的一个重要情况是，对象作为参数传递给从Python调用的扩展模块中的C函数;调用机制保证在调用期间保持对每个参数的引用。
<br/>

然而，一个常见的缺陷是从列表中提取一个对象并保留它一段时间，而不增加它的引用计数。其他一些操作可能会从列表中删除对象，减少它的引用计数，并可能释放它。真正的危险在于，看似无害的操作可能会调用能够做到这一点的任意Python代码;有一个代码路径允许控制从`Py_DECREF()`返回到用户，所以几乎任何操作都有潜在危险。
<br/>

安全的方法是始终使用泛型操作(名称以PyObject_、PyNumber_、PySequence_或PyMapping_开头的函数)。这些操作总是增加它们返回的对象的引用计数。这使得调用方有责任在处理完结果后调用`Py_DECREF()`。

#### 引用计数细节

`Python/C` API中函数的引用计数行为最好从引用所有权的角度来解释。所有权属于引用，而不是对象(对象是不拥有的:它们总是共享的)。“拥有一个引用”意味着当该引用不再需要时，负责对其调用`Py_DECREF`。所有权也可以转移，这意味着接收引用所有权的代码将负责在不再需要它时通过调用`Py_DECREF()`或`Py_XDECREF()`来最终减少它——或者将此责任传递(通常传递给其调用者)。当函数将引用的所有权传递给它的调用者时，就称调用者收到了一个新引用。当没有所有权转移时，调用者被称为借用了引用。不需要为借用的参考文献做什么。
<br/>

相反，当调用函数传入对象的引用时，有两种可能:函数窃取对象的引用，或者没有。窃取引用意味着当您向一个函数传递引用时，该函数假定它现在拥有该引用，而您不再对它负责。
<br/>

很少有函数窃取引用;两个值得注意的例外是`PyList_SetItem()`和`PyTuple_SetItem()`，它们会窃取对条目的引用(但不会窃取条目所在的元组或列表!)这些函数被设计为窃取引用，这是因为使用新创建的对象填充元组或列表的常见习惯;例如，创建元组(1,2，“3”)的代码可能是这样的(暂时忘记错误处理;下面是一种更好的编码方式):
```c
PyObject *t;

t = PyTuple_New(3);
PyTuple_SetItem(t, 0, PyLong_FromLong(1L));
PyTuple_SetItem(t, 1, PyLong_FromLong(2L));
PyTuple_SetItem(t, 2, PyUnicode_FromString("three"));
```

在这里，`PyLong_FromLong()`返回一个新的引用，该引用立即被`PyTuple_SetItem()`窃取。当您想继续使用一个对象，尽管它的引用将被窃取时，请在调用引用窃取函数之前使用`Py_INCREF()`获取另一个引用。
<br/>

顺便说一句，`PyTuple_SetItem()`是设置元组项的唯一方法;`PySequence_SetItem()`和`PyObject_SetItem()`拒绝这样做，因为元组是不可变的数据类型。你应该只对你自己创建的元组使用`PyTuple_SetItem()`。
<br/>

可以使用`PyList_New()`和`PyList_SetItem()`编写填充列表的等效代码。
<br/>

然而，在实践中，您很少使用这些方法来创建和填充元组或列表。有一个泛型函数`Py_BuildValue()`，它可以通过格式字符串从C值创建最常见的对象。例如，上面的两个代码块可以被以下代码替换(它也负责错误检查):
```c
PyObject *tuple, *list;

tuple = Py_BuildValue("(iis)", 1, 2, "three");
list = Py_BuildValue("[iis]", 1, 2, "three");
```

更常见的做法是使用`PyObject_SetItem()`和友项，它们的引用只是借用，比如传递给正在编写的函数的参数。在这种情况下，它们关于引用计数的行为更合理，因为你不必增加引用计数，所以你可以给出一个引用(“让它被窃取”)。例如，这个函数将列表中的所有项(实际上是任何可变序列)设置为给定项:
```c
int set_all(PyObject *target, PyObject *item)
{
    Py_ssize_t i, n;

    n = PyObject_Length(target);
    if (n < 0)
        return -1;
    for (i = 0; i < n; i++) {
        PyObject *index = PyLong_FromSsize_t(i);
        if (!index)
            return -1;
        if (PyObject_SetItem(target, index, item) < 0) {
            Py_DECREF(index);
            return -1;
        }
        Py_DECREF(index);
    }
    return 0;
}
```
<br/>

函数返回值的情况略有不同。虽然将引用传递给大多数函数不会改变您对该引用的所有权责任，但许多返回对象引用的函数会给您该引用的所有权。原因很简单:在许多情况下，返回的对象是动态创建的，而您获得的引用是对该对象的唯一引用。因此，返回对象引用的泛型函数，如`PyObject_GetItem()`和`PySequence_GetItem()`，总是返回一个新的引用(调用者成为引用的所有者)。
<br/>

重要的是要意识到，你是否拥有一个函数返回的引用取决于你只调用了哪个函数，如果使用`PyList_GetItem()`从列表中提取一个条目，则不拥有该引用——但如果使用`PySequence_GetItem()`从相同的列表中获得相同的条目(它恰好接受完全相同的参数)，则拥有对返回对象的引用。
<br/>

下面是一个例子，你可以编写一个函数来计算整数列表中各项的和;一次使用`PyList_GetItem()`，一次使用`PySequence_GetItem()`。
```c
long sum_list(PyObject *list)
{
    Py_ssize_t i, n;
    long total = 0, value;
    PyObject *item;

    n = PyList_Size(list);
    if (n < 0)
        return -1; /* Not a list */
    for (i = 0; i < n; i++) {
        item = PyList_GetItem(list, i); /* Can't fail */
        if (!PyLong_Check(item)) continue; /* Skip non-integers */
        value = PyLong_AsLong(item);
        if (value == -1 && PyErr_Occurred())
            /* Integer too big to fit in a C long, bail out */
            return -1;
        total += value;
    }
    return total;
}
```

```c
long sum_sequence(PyObject *sequence)
{
    Py_ssize_t i, n;
    long total = 0, value;
    PyObject *item;
    n = PySequence_Length(sequence);
    if (n < 0)
        return -1; /* Has no length */
    for (i = 0; i < n; i++) {
        item = PySequence_GetItem(sequence, i);
        if (item == NULL)
            return -1; /* Not a sequence, or other failure */
        if (PyLong_Check(item)) {
            value = PyLong_AsLong(item);
            Py_DECREF(item);
            if (value == -1 && PyErr_Occurred())
                /* Integer too big to fit in a C long, bail out */
                return -1;
            total += value;
        } else {
            Py_DECREF(item); /* Discard reference ownership */
        }
    }
    return total;
}
```

#### 类型

很少有其他数据类型在`Python/C` API中扮演重要角色;大多数是简单的C类型，如`int`、`long`、`double`和`char*`。一些结构类型用于描述静态表，用于列出模块导出的函数或新对象类型的数据属性，另一种结构类型用于描述复数的值。这些将与使用它们的函数一起讨论。

`type Py_ssize_t`

一种有符号整型，其`sizeof(Py_ssize_t) == sizeof(size_t)`。`C99`没有直接定义这种类型(`size_t`是无符号整型)。详见PEP 353。`PY_SSIZE_T_MAX`是`Py_ssize_t`类型的最大正值。

#### 异常

Python程序员只有在需要特定错误处理时才需要处理异常;未处理的异常会自动传播到调用者，然后传播到调用者的调用者，依此类推，直到它们到达顶级解释器，在那里它们会被报告给用户，并伴随堆栈回溯。
<br/>

然而，对于C程序员来说，错误检查总是必须是显式的。`Python/C` API中的所有函数都可能引发异常，除非在函数文档中另有显式声明。通常，当函数遇到错误时，它会设置一个异常，丢弃它所拥有的任何对象引用，并返回一个错误指示器。如果没有其他说明，则该指示符为`NULL`或`-1`，这取决于函数的返回类型。一些函数返回布尔值的`true/false`结果，其中false表示错误。很少有函数不返回显式的错误指示符或返回值不明确，并且需要使用`PyErr_Occurred()`显式测试错误。这些异常总是显式地记录下来。
<br/>

异常状态在每个线程存储中维护(这相当于在非线程应用程序中使用全局存储)。线程可以处于两种状态之一:发生了异常，或者没有发生异常。函数`pyerr_occurs()`可用于检查:当异常发生时，它返回一个借用的异常类型对象的引用，否则返回`NULL`。有许多函数可以设置异常状态:`PyErr_SetString()`是最常见的(尽管不是最通用的)设置异常状态的函数，`PyErr_Clear()`则清除异常状态。
<br/>

完整的异常状态由三个对象组成(所有对象都可以是`NULL`):`异常类型`、`对应的异常值`和`回溯`。它们与`sys.exc_info()`的Python结果具有相同的含义;然而，它们并不相同:`Python`对象表示由`Python try…except`语句处理的最后一个异常，而C级异常状态只在异常在C函数之间传递时才存在，直到它到达Python字节码解释器的主循环，由解释器负责将其传递给`sys.exc_info()`和`friends`。
<br/>

请注意，从`Python 1.5`开始，从Python代码中访问异常状态的首选的、线程安全的方法是调用`sys.exc_info()`函数，该函数会为`Python`代码返回每个线程的异常状态。此外，访问异常状态的两种方法的语义也发生了变化，因此捕获异常的函数将保存并恢复其线程的异常状态，以便保留其调用方的异常状态。这可以防止异常处理代码中由一个看起来无害的函数覆盖正在处理的异常而引起的常见bug;它还减少了回溯中堆栈帧引用的对象通常不希望看到的生存期扩展。
<br/>

作为一般原则，调用另一个函数来执行某些任务的函数应该检查被调用函数是否引发异常，如果是，则将异常状态传递给调用者。它应该丢弃它所拥有的任何对象引用，并返回一个错误指示符，但它不应该设置另一个异常—那将覆盖刚刚引发的异常，并丢失关于错误的确切原因的重要信息。
<br/>

上面的`sum_sequence()`示例显示了一个检测异常并将其传递下去的简单示例。因此，当本例检测到错误时，它不需要清除任何拥有的引用。下面的示例函数显示了一些错误清除。首先，为了提醒你为什么喜欢Python，我们展示了等价的Python代码:
```python
def incr_item(dict, key):
    try:
        item = dict[key]
    except KeyError:
        item = 0
    dict[key] = item + 1
```
下面是相应的C代码:
```c
int incr_item(PyObject *dict, PyObject *key)
{
    /* Objects all initialized to NULL for Py_XDECREF */
    PyObject *item = NULL, *const_one = NULL, *incremented_item = NULL;
    int rv = -1; /* Return value initialized to -1 (failure) */

    item = PyObject_GetItem(dict, key);
    if (item == NULL) {
        /* Handle KeyError only: */
        if (!PyErr_ExceptionMatches(PyExc_KeyError))
            goto error;

        /* Clear the error and use zero: */
        PyErr_Clear();
        item = PyLong_FromLong(0L);
        if (item == NULL)
            goto error;
    }
    const_one = PyLong_FromLong(1L);
    if (const_one == NULL)
        goto error;

    incremented_item = PyNumber_Add(item, const_one);
    if (incremented_item == NULL)
        goto error;

    if (PyObject_SetItem(dict, key, incremented_item) < 0)
        goto error;
    rv = 0; /* Success */
    /* Continue with cleanup code */

 error:
    /* Cleanup code, shared by success and failure path */

    /* Use Py_XDECREF() to ignore NULL references */
    Py_XDECREF(item);
    Py_XDECREF(const_one);
    Py_XDECREF(incremented_item);

    return rv; /* -1 for error, 0 for success */
}

```

这个例子代表了C语言中goto语句的认可使用!它说明了如何使用`PyErr_ExceptionMatches()`和`PyErr_Clear()`来处理特定的异常，以及如何使用`Py_XDECREF()`来处理可能为`NULL`的引用(注意名称中的'X';当遇到NULL引用时，`Py_DECREF()`会崩溃)。重要的是，用于保存所有引用的变量被初始化为`NULL`，这样才能工作;同样，建议的返回值被初始化为`-1`(失败)，只有在最后一次调用成功后才设置为成功。

### 嵌入Python

另外Python解释器的嵌入人员(而不是扩展编写人员)需要担心的一项重要任务是Python解释器的初始化和销毁。解释器的大多数功能只能在解释器初始化之后才能使用。
<br/>

基本初始化函数是`Py_Initialize()`。这会初始化已加载模块的表，并创建基本模块`builtins`、`__main__`和`sys`。它还初始化模块搜索路径(`sys.path`)。
<br/>

`Py_Initialize()`没有设置“脚本参数列表”(`sys.argv`)。如果稍后执行的Python代码需要此变量，则必须在调用`Py_Initialize()`之后通过调用`PySys_SetArgvEx(argc, argv, updatepath)`显式地设置它。
<br/>

在大多数系统上(特别是在`Unix`和`Windows上`，尽管细节略有不同)，`Py_Initialize()`根据其对标准Python解释器可执行文件位置的最佳猜测来计算模块搜索路径，假设`Python`库是在相对于Python解释器可执行文件的固定位置找到的。特别地，它查找名为`lib/pythonX`的目录。Y相对于shell命令搜索路径(环境变量path)中可执行文件python所在的父目录。
<br/>

例如，如果Python可执行文件在`/usr/local/bin/python`中找到，它将假定库在`/usr/local/lib/pythonx.y`中(事实上，这个特定的路径也是“回退”位置，当沿着path找不到名为python的可执行文件时使用。)用户可以通过设置环境变量`PYTHONHOME`来覆盖此行为，或者通过设置`PYTHONPATH`在标准路径前面插入额外的目录。
<br/>

嵌入应用程序可以通过在调用`Py_Initialize()`之前调用`Py_SetProgramName(file)`来控制搜索。请注意，`PYTHONHOME`仍然会重写此内容，并且`PYTHONPATH`仍然会插入到标准路径的前面。一个需要完全控制的应用程序必须提供它自己的`Py_GetPath()`、`Py_GetPrefix()`、`Py_GetExecPrefix()`和`Py_GetProgramFullPath()`的实现(所有定义在`Modules/getpath.c`中)。
<br/>

有时，“反初始化”Python是可取的。例如，应用程序可能想重新开始(再次调用`Py_Initialize()`)，或者应用程序只是简单地使用了Python并希望释放Python分配的内存。这可以通过调用`Py_FinalizeEx()`来完成。如果Python当前处于初始化状态，则函数`Py_IsInitialized()`返回true。关于这些函数的更多信息将在后面的章节中给出。注意`Py_FinalizeEx()`不会释放Python解释器分配的所有内存，例如由扩展模块分配的内存目前无法释放。

### 调试构建

Python可以使用几个宏来构建，以支持对解释器和扩展模块的额外检查。这些检查往往会给运行时增加大量开销，因此默认情况下不会启用它们。
<br/>

各种类型的调试构建的完整列表在Python源代码发行版中的`Misc/SpecialBuilds.txt`文件中。提供了支持跟踪引用计数、调试内存分配器或主解释器循环的低级分析的版本。本节的其余部分将只描述最常用的构建。
<br/>

使用定义的`Py_DEBUG`宏编译解释器会产生通常意义上的Python调试构建。通过在`./configure`命令中添加`——with-pydebug`，可以在`Unix`构建中启用`Py_DEBUG`。不特定于`python`的`_DEBUG`宏的存在也暗示了这一点。当在Unix编译中启用`Py_DEBUG`时，将禁用编译器优化。
<br/>

除了下面描述的引用计数调试之外，还会执行额外的检查，请参阅Python调试构建。
<br/>

定义`Py_TRACE_REFS`启用引用跟踪(请参阅配置`——with-trace-refs`选项)。定义后，活动对象的循环双链接列表通过向每个`PyObject`添加两个额外字段来维护。总分配也被跟踪。退出时，将打印所有现有的引用。(在交互模式下，这发生在解释器运行的每条语句之后。)
<br/>

请参阅Python源代码中的`Misc/SpecialBuilds.txt`获取更多详细信息。

## C API

Python的C API被向后兼容策略PEP 387覆盖。虽然C API会随着每一个小版本而变化(例如从3.9到3.10)，但大多数变化都是源代码兼容的，通常只会添加新的API。只有在弃用期过后或修复严重问题时才会更改现有API或删除API。
<br/>

CPython的应用程序二进制接口(ABI)在一个小版本中是向前和向后兼容的(如果它们以相同的方式编译;请参阅下面的平台注意事项)。因此，为Python 3.10.0编译的代码将在3.10.8上运行，反之亦然，但需要为单独编译3.9.x和3.10.x。
<br/>

以下划线作为前缀的名称(如`_Py_InternalState`)是私有API，即使在补丁版本中也可以不经通知地更改。

### 程序二进制接口

Python 3.2引入了Limited API，它是Python C API的一个子集。只使用Limited API的扩展可以被编译一次并使用多个版本的Python。Limited API的内容如下。
<br/>

为了实现这一点，Python提供了一个稳定ABI: 一组将在Python 3.x中保持兼容的符号版本。Stable ABI包含了Limited API中公开的符号，但也包含了其他的符号——例如，支持老版本Limited API所必需的函数。

> 为了简单起见，本文讨论了扩展，但是有限API和稳定ABI对于API的所有使用都以相同的方式工作——例如：Python嵌入其它程序的使用。

### 宏: `Py_LIMITED_API`

在包含Python.h之前定义此宏以选择只使用有限API，并选择有限API版本
<br/>

将`Py_LIMITED_API`定义为`PY_VERSION_HEX`的值，该值对应于扩展支持的最低Python版本。该扩展不需要对指定版本之后的所有Python 3版本进行重新编译，并且可以使用该版本之前引入的Limited API。
<br/>

与其直接使用`PY_VERSION_HEX`宏，不如硬编码一个最小的次要版本(例如`Python 3.10`的版本为`0x030A0000`)，以确保在使用未来的Python版本编译时的稳定性。
<br/>

您还可以将`Py_LIMITED_API`定义为3。这与`0x03020000` (`Python 3.2`，引入了Limited API的版本)的工作原理相同。
<br/>

在Windows上，使用稳定ABI的扩展应该链接到`python3.dll`，而不是特定于版本的库，如`python39.dll`。
<br/>

在某些平台上，Python会查找并加载以`abi3`标签命名的共享库文件(例如`mymodule.abi3.so`)。它不检查这样的扩展是否符合Stable ABI。例如，用户(或其打包工具)需要确保使用`3.10+` Limited API构建的扩展不会为较低版本的Python安装。
<br/>

Stable ABI中的所有函数都以Python共享库中的函数形式存在，而不仅仅是宏。这使得它们在不使用C预处理器的语言中可用。

### `Limited API` 的范围和性能

有限API的目标是允许使用完整的C API，但可能会有性能上的损失。
<br/>

例如，虽然`PyList_GetItem()`可用，但它的“不安全”宏变体`PyList_GET_ITEM()`不可用。宏可以更快，因为它可以依赖于列表对象的特定于版本的实现细节。
<br/>

如果没有定义`Py_LIMITED_API`，一些`C` `API`函数会内联或被宏取代。定义`Py_LIMITED_API`禁用了这种内联，允许在改进Python数据结构时保持稳定性，但可能会降低性能。
<br/>

通过省略`Py_LIMITED_API`定义，可以使用特定于版本的ABI来编译一个`Limited API`扩展。这可以提高该Python版本的性能，但会限制兼容性。使用`Py_LIMITED_API`编译将产生一个扩展，该扩展可以在特定于版本的扩展不可用的地方发布—例如，用于即将发布的Python版本的预发行版。
<br/>

#### `Limited API` 注意事项

注意，使用`Py_LIMITED_API`编译并不能完全保证代码符合有限API或稳定ABI。`Py_LIMITED_API`只涵盖定义，但API还包括其他问题，如预期语义。
<br/>

`Py_LIMITED_API`无法防范的一个问题是，在较低的Python版本中调用带有无效参数的函数。例如，假设一个函数开始接受`NULL`作为参数。在Python 3.9中，NULL现在选择默认行为，但在Python 3.8中，参数将被直接使用，导致NULL解引用和崩溃。类似的参数也适用于结构的字段。
<br/>

另一个问题是，当定义`Py_LIMITED_API`时，一些struct字段目前不隐藏，即使它们是Limited API的一部分。
<br/>

出于这些原因，我们建议使用它支持的所有次要Python版本测试扩展，并最好使用最低的此类版本进行构建。
<br/>

我们还建议查看所有使用的API的文档，以检查它是否明确属于`Limited API`的一部分。即使定义了`Py_LIMITED_API`，一些私有声明也会因为技术原因被公开(甚至是无意中作为bug)。
<br/>

还要注意，`Limited API`并不一定是稳定的:在`Python 3.8`中使用`Py_LIMITED_API`编译意味着扩展将在`Python 3.12`中运行，但它不一定会在`Python 3.12`中编译。特别地，如果稳定ABI保持稳定，有限API的部分可能会被弃用和删除。

#### 平台考虑

ABI的稳定性不仅依赖于Python，还依赖于所使用的编译器、底层库和编译器选项。对于稳定ABI来说，这些细节定义了一个“平台”。它们通常取决于操作系统类型和处理器架构。
<br/>

每个特定的Python分发者都有责任确保特定平台上的所有Python版本以不破坏稳定ABI的方式构建。python.org和许多第三方分销商发布的Windows和macOS版本就是这样。

### `Limited API`的内容

目前，`Limited API`包括以下项目:

#### `PyAIter_Check()`
#### `PyArg_Parse()`
#### `PyArg_ParseTuple()`
#### `PyArg_ParseTupleAndKeywords()`
#### `PyArg_UnpackTuple()`
#### `PyArg_VaParse()`
#### `PyArg_VaParseTupleAndKeywords()`
#### `PyArg_ValidateKeywordArguments()`
#### `PyBaseObject_Type`
#### `PyBool_FromLong()`
#### `PyBool_Type`
#### `PyByteArrayIter_Type`
#### `PyByteArray_AsString()`
#### `PyByteArray_Concat()`
#### `PyByteArray_FromObject()`
#### `PyByteArray_FromStringAndSize()`
#### `PyByteArray_Resize()`
#### `PyByteArray_Size()`
#### `PyByteArray_Type`
#### `PyBytesIter_Type`
#### `PyBytes_AsString()`
#### `PyBytes_AsStringAndSize()`
#### `PyBytes_Concat()`
#### `PyBytes_ConcatAndDel()`
#### `PyBytes_DecodeEscape()`
#### `PyBytes_FromFormat()`
#### `PyBytes_FromFormatV()`
#### `PyBytes_FromObject()`
#### `PyBytes_FromString()`
#### `PyBytes_FromStringAndSize()`
#### `PyBytes_Repr()`
#### `PyBytes_Size()`
#### `PyBytes_Type`
#### `PyCFunction`
#### `PyCFunctionWithKeywords`
#### `PyCFunction_Call()`
#### `PyCFunction_GetFlags()`
#### `PyCFunction_GetFunction()`
#### `PyCFunction_GetSelf()`
#### `PyCFunction_New()`
#### `PyCFunction_NewEx()`
#### `PyCFunction_Type`
#### `PyCMethod_New()`
#### `PyCallIter_New()`
#### `PyCallIter_Type`
#### `PyCallable_Check()`
#### `PyCapsule_Destructor`
#### `PyCapsule_GetContext()`
#### `PyCapsule_GetDestructor()`
#### `PyCapsule_GetName()`
#### `PyCapsule_GetPointer()`
#### `PyCapsule_Import()`
#### `PyCapsule_IsValid()`
#### `PyCapsule_New()`
#### `PyCapsule_SetContext()`
#### `PyCapsule_SetDestructor()`
#### `PyCapsule_SetName()`
#### `PyCapsule_SetPointer()`
#### `PyCapsule_Type`
#### `PyClassMethodDescr_Type`
#### `PyCodec_BackslashReplaceErrors()`
#### `PyCodec_Decode()`
#### `PyCodec_Decoder()`
#### `PyCodec_Encode()`
#### `PyCodec_Encoder()`
#### `PyCodec_IgnoreErrors()`
#### `PyCodec_IncrementalDecoder()`
#### `PyCodec_IncrementalEncoder()`
#### `PyCodec_KnownEncoding()`
#### `PyCodec_LookupError()`
#### `PyCodec_NameReplaceErrors()`
#### `PyCodec_Register()`
#### `PyCodec_RegisterError()`
#### `PyCodec_ReplaceErrors()`
#### `PyCodec_StreamReader()`
#### `PyCodec_StreamWriter()`
#### `PyCodec_StrictErrors()`
#### `PyCodec_Unregister()`
#### `PyCodec_XMLCharRefReplaceErrors()`
#### `PyComplex_FromDoubles()`
#### `PyComplex_ImagAsDouble()`
#### `PyComplex_RealAsDouble()`
#### `PyComplex_Type`
#### `PyDescr_NewClassMethod()`
#### `PyDescr_NewGetSet()`
#### `PyDescr_NewMember()`
#### `PyDescr_NewMethod()`
#### `PyDictItems_Type`
#### `PyDictIterItem_Type`
#### `PyDictIterKey_Type`
#### `PyDictIterValue_Type`
#### `PyDictKeys_Type`
#### `PyDictProxy_New()`
#### `PyDictProxy_Type`
#### `PyDictRevIterItem_Type`
#### `PyDictRevIterKey_Type`
#### `PyDictRevIterValue_Type`
#### `PyDictValues_Type`
#### `PyDict_Clear()`
#### `PyDict_Contains()`
#### `PyDict_Copy()`
#### `PyDict_DelItem()`
#### `PyDict_DelItemString()`
#### `PyDict_GetItem()`
#### `PyDict_GetItemString()`
#### `PyDict_GetItemWithError()`
#### `PyDict_Items()`
#### `PyDict_Keys()`
#### `PyDict_Merge()`
#### `PyDict_MergeFromSeq2()`
#### `PyDict_New()`
#### `PyDict_Next()`
#### `PyDict_SetItem()`
#### `PyDict_SetItemString()`
#### `PyDict_Size()`
#### `PyDict_Type`
#### `PyDict_Update()`
#### `PyDict_Values()`
#### `PyEllipsis_Type`
#### `PyEnum_Type`
#### `PyErr_BadArgument()`
#### `PyErr_BadInternalCall()`
#### `PyErr_CheckSignals()`
#### `PyErr_Clear()`
#### `PyErr_Display()`
#### `PyErr_ExceptionMatches()`
#### `PyErr_Fetch()`
#### `PyErr_Format()`
#### `PyErr_FormatV()`
#### `PyErr_GetExcInfo()`
#### `PyErr_GivenExceptionMatches()`
#### `PyErr_NewException()`
#### `PyErr_NewExceptionWithDoc()`
#### `PyErr_NoMemory()`
#### `PyErr_NormalizeException()`
#### `PyErr_Occurred()`
#### `PyErr_Print()`
#### `PyErr_PrintEx()`
#### `PyErr_ProgramText()`
#### `PyErr_ResourceWarning()`
#### `PyErr_Restore()`
#### `PyErr_SetExcFromWindowsErr()`
#### `PyErr_SetExcFromWindowsErrWithFilename()`
#### `PyErr_SetExcFromWindowsErrWithFilenameObject()`
#### `PyErr_SetExcFromWindowsErrWithFilenameObjects()`
#### `PyErr_SetExcInfo()`
#### `PyErr_SetFromErrno()`
#### `PyErr_SetFromErrnoWithFilename()`
#### `PyErr_SetFromErrnoWithFilenameObject()`
#### `PyErr_SetFromErrnoWithFilenameObjects()`
#### `PyErr_SetFromWindowsErr()`
#### `PyErr_SetFromWindowsErrWithFilename()`
#### `PyErr_SetImportError()`
#### `PyErr_SetImportErrorSubclass()`
#### `PyErr_SetInterrupt()`
#### `PyErr_SetInterruptEx()`
#### `PyErr_SetNone()`
#### `PyErr_SetObject()`
#### `PyErr_SetString()`
#### `PyErr_SyntaxLocation()`
#### `PyErr_SyntaxLocationEx()`
#### `PyErr_WarnEx()`
#### `PyErr_WarnExplicit()`
#### `PyErr_WarnFormat()`
#### `PyErr_WriteUnraisable()`
#### `PyEval_AcquireLock()`
#### `PyEval_AcquireThread()`
#### `PyEval_CallFunction()`
#### `PyEval_CallMethod()`
#### `PyEval_CallObjectWithKeywords()`
#### `PyEval_EvalCode()`
#### `PyEval_EvalCodeEx()`
#### `PyEval_EvalFrame()`
#### `PyEval_EvalFrameEx()`
#### `PyEval_GetBuiltins()`
#### `PyEval_GetFrame()`
#### `PyEval_GetFuncDesc()`
#### `PyEval_GetFuncName()`
#### `PyEval_GetGlobals()`
#### `PyEval_GetLocals()`
#### `PyEval_InitThreads()`
#### `PyEval_ReleaseLock()`
#### `PyEval_ReleaseThread()`
#### `PyEval_RestoreThread()`
#### `PyEval_SaveThread()`
#### `PyEval_ThreadsInitialized()`
#### `PyExc_ArithmeticError`
#### `PyExc_AssertionError`
#### `PyExc_AttributeError`
#### `PyExc_BaseException`
#### `PyExc_BlockingIOError`
#### `PyExc_BrokenPipeError`
#### `PyExc_BufferError`
#### `PyExc_BytesWarning`
#### `PyExc_ChildProcessError`
#### `PyExc_ConnectionAbortedError`
#### `PyExc_ConnectionError`
#### `PyExc_ConnectionRefusedError`
#### `PyExc_ConnectionResetError`
#### `PyExc_DeprecationWarning`
#### `PyExc_EOFError`
#### `PyExc_EncodingWarning`
#### `PyExc_EnvironmentError`
#### `PyExc_Exception`
#### `PyExc_FileExistsError`
#### `PyExc_FileNotFoundError`
#### `PyExc_FloatingPointError`
#### `PyExc_FutureWarning`
#### `PyExc_GeneratorExit`
#### `PyExc_IOError`
#### `PyExc_ImportError`
#### `PyExc_ImportWarning`
#### `PyExc_IndentationError`
#### `PyExc_IndexError`
#### `PyExc_InterruptedError`
#### `PyExc_IsADirectoryError`
#### `PyExc_KeyError`
#### `PyExc_KeyboardInterrupt`
#### `PyExc_LookupError`
#### `PyExc_MemoryError`
#### `PyExc_ModuleNotFoundError`
#### `PyExc_NameError`
#### `PyExc_NotADirectoryError`
#### `PyExc_NotImplementedError`
#### `PyExc_OSError`
#### `PyExc_OverflowError`
#### `PyExc_PendingDeprecationWarning`
#### `PyExc_PermissionError`
#### `PyExc_ProcessLookupError`
#### `PyExc_RecursionError`
#### `PyExc_ReferenceError`
#### `PyExc_ResourceWarning`
#### `PyExc_RuntimeError`
#### `PyExc_RuntimeWarning`
#### `PyExc_StopAsyncIteration`
#### `PyExc_StopIteration`
#### `PyExc_SyntaxError`
#### `PyExc_SyntaxWarning`
#### `PyExc_SystemError`
#### `PyExc_SystemExit`
#### `PyExc_TabError`
#### `PyExc_TimeoutError`
#### `PyExc_TypeError`
#### `PyExc_UnboundLocalError`
#### `PyExc_UnicodeDecodeError`
#### `PyExc_UnicodeEncodeError`
#### `PyExc_UnicodeError`
#### `PyExc_UnicodeTranslateError`
#### `PyExc_UnicodeWarning`
#### `PyExc_UserWarning`
#### `PyExc_ValueError`
#### `PyExc_Warning`
#### `PyExc_WindowsError`
#### `PyExc_ZeroDivisionError`
#### `PyExceptionClass_Name()`
#### `PyException_GetCause()`
#### `PyException_GetContext()`
#### `PyException_GetTraceback()`
#### `PyException_SetCause()`
#### `PyException_SetContext()`
#### `PyException_SetTraceback()`
#### `PyFile_FromFd()`
#### `PyFile_GetLine()`
#### `PyFile_WriteObject()`
#### `PyFile_WriteString()`
#### `PyFilter_Type`
#### `PyFloat_AsDouble()`
#### `PyFloat_FromDouble()`
#### `PyFloat_FromString()`
#### `PyFloat_GetInfo()`
#### `PyFloat_GetMax()`
#### `PyFloat_GetMin()`
#### `PyFloat_Type`
#### `PyFrameObject`
#### `PyFrame_GetCode()`
#### `PyFrame_GetLineNumber()`
#### `PyFrozenSet_New()`
#### `PyFrozenSet_Type`
#### `PyGC_Collect()`
#### `PyGC_Disable()`
#### `PyGC_Enable()`
#### `PyGC_IsEnabled()`
#### `PyGILState_Ensure()`
#### `PyGILState_GetThisThreadState()`
#### `PyGILState_Release()`
#### `PyGILState_STATE`
#### `PyGetSetDef`
#### `PyGetSetDescr_Type`
#### `PyImport_AddModule()`
#### `PyImport_AddModuleObject()`
#### `PyImport_AppendInittab()`
#### `PyImport_ExecCodeModule()`
#### `PyImport_ExecCodeModuleEx()`
#### `PyImport_ExecCodeModuleObject()`
#### `PyImport_ExecCodeModuleWithPathnames()`
#### `PyImport_GetImporter()`
#### `PyImport_GetMagicNumber()`
#### `PyImport_GetMagicTag()`
#### `PyImport_GetModule()`
#### `PyImport_GetModuleDict()`
#### `PyImport_Import()`
#### `PyImport_ImportFrozenModule()`
#### `PyImport_ImportFrozenModuleObject()`
#### `PyImport_ImportModule()`
#### `PyImport_ImportModuleLevel()`
#### `PyImport_ImportModuleLevelObject()`
#### `PyImport_ImportModuleNoBlock()`
#### `PyImport_ReloadModule()`
#### `PyIndex_Check()`
#### `PyInterpreterState`
#### `PyInterpreterState_Clear()`
#### `PyInterpreterState_Delete()`
#### `PyInterpreterState_Get()`
#### `PyInterpreterState_GetDict()`
#### `PyInterpreterState_GetID()`
#### `PyInterpreterState_New()`
#### `PyIter_Check()`
#### `PyIter_Next()`
#### `PyIter_Send()`
#### `PyListIter_Type`
#### `PyListRevIter_Type`
#### `PyList_Append()`
#### `PyList_AsTuple()`
#### `PyList_GetItem()`
#### `PyList_GetSlice()`
#### `PyList_Insert()`
#### `PyList_New()`
#### `PyList_Reverse()`
#### `PyList_SetItem()`
#### `PyList_SetSlice()`
#### `PyList_Size()`
#### `PyList_Sort()`
#### `PyList_Type`
#### `PyLongObject`
#### `PyLongRangeIter_Type`
#### `PyLong_AsDouble()`
#### `PyLong_AsLong()`
#### `PyLong_AsLongAndOverflow()`
#### `PyLong_AsLongLong()`
#### `PyLong_AsLongLongAndOverflow()`
#### `PyLong_AsSize_t()`
#### `PyLong_AsSsize_t()`
#### `PyLong_AsUnsignedLong()`
#### `PyLong_AsUnsignedLongLong()`
#### `PyLong_AsUnsignedLongLongMask()`
#### `PyLong_AsUnsignedLongMask()`
#### `PyLong_AsVoidPtr()`
#### `PyLong_FromDouble()`
#### `PyLong_FromLong()`
#### `PyLong_FromLongLong()`
#### `PyLong_FromSize_t()`
#### `PyLong_FromSsize_t()`
#### `PyLong_FromString()`
#### `PyLong_FromUnsignedLong()`
#### `PyLong_FromUnsignedLongLong()`
#### `PyLong_FromVoidPtr()`
#### `PyLong_GetInfo()`
#### `PyLong_Type`
#### `PyMap_Type`
#### `PyMapping_Check()`
#### `PyMapping_GetItemString()`
#### `PyMapping_HasKey()`
#### `PyMapping_HasKeyString()`
#### `PyMapping_Items()`
#### `PyMapping_Keys()`
#### `PyMapping_Length()`
#### `PyMapping_SetItemString()`
#### `PyMapping_Size()`
#### `PyMapping_Values()`
#### `PyMem_Calloc()`
#### `PyMem_Free()`
#### `PyMem_Malloc()`
#### `PyMem_Realloc()`
#### `PyMemberDef`
#### `PyMemberDescr_Type`
#### `PyMemoryView_FromMemory()`
#### `PyMemoryView_FromObject()`
#### `PyMemoryView_GetContiguous()`
#### `PyMemoryView_Type`
#### `PyMethodDef`
#### `PyMethodDescr_Type`
#### `PyModuleDef`
#### `PyModuleDef_Base`
#### `PyModuleDef_Init()`
#### `PyModuleDef_Type`
#### `PyModule_AddFunctions()`
#### `PyModule_AddIntConstant()`
#### `PyModule_AddObject()`
#### `PyModule_AddObjectRef()`
#### `PyModule_AddStringConstant()`
#### `PyModule_AddType()`
#### `PyModule_Create2()`
#### `PyModule_ExecDef()`
#### `PyModule_FromDefAndSpec2()`
#### `PyModule_GetDef()`
#### `PyModule_GetDict()`
#### `PyModule_GetFilename()`
#### `PyModule_GetFilenameObject()`
#### `PyModule_GetName()`
#### `PyModule_GetNameObject()`
#### `PyModule_GetState()`
#### `PyModule_New()`
#### `PyModule_NewObject()`
#### `PyModule_SetDocString()`
#### `PyModule_Type`
#### `PyNumber_Absolute()`
#### `PyNumber_Add()`
#### `PyNumber_And()`
#### `PyNumber_AsSsize_t()`
#### `PyNumber_Check()`
#### `PyNumber_Divmod()`
#### `PyNumber_Float()`
#### `PyNumber_FloorDivide()`
#### `PyNumber_InPlaceAdd()`
#### `PyNumber_InPlaceAnd()`
#### `PyNumber_InPlaceFloorDivide()`
#### `PyNumber_InPlaceLshift()`
#### `PyNumber_InPlaceMatrixMultiply()`
#### `PyNumber_InPlaceMultiply()`
#### `PyNumber_InPlaceOr()`
#### `PyNumber_InPlacePower()`
#### `PyNumber_InPlaceRemainder()`
#### `PyNumber_InPlaceRshift()`
#### `PyNumber_InPlaceSubtract()`
#### `PyNumber_InPlaceTrueDivide()`
#### `PyNumber_InPlaceXor()`
#### `PyNumber_Index()`
#### `PyNumber_Invert()`
#### `PyNumber_Long()`
#### `PyNumber_Lshift()`
#### `PyNumber_MatrixMultiply()`
#### `PyNumber_Multiply()`
#### `PyNumber_Negative()`
#### `PyNumber_Or()`
#### `PyNumber_Positive()`
#### `PyNumber_Power()`
#### `PyNumber_Remainder()`
#### `PyNumber_Rshift()`
#### `PyNumber_Subtract()`
#### `PyNumber_ToBase()`
#### `PyNumber_TrueDivide()`
#### `PyNumber_Xor()`
#### `PyOS_AfterFork()`
#### `PyOS_AfterFork_Child()`
#### `PyOS_AfterFork_Parent()`
#### `PyOS_BeforeFork()`
#### `PyOS_CheckStack()`
#### `PyOS_FSPath()`
#### `PyOS_InputHook`
#### `PyOS_InterruptOccurred()`
#### `PyOS_double_to_string()`
#### `PyOS_getsig()`
#### `PyOS_mystricmp()`
#### `PyOS_mystrnicmp()`
#### `PyOS_setsig()`
#### `PyOS_sighandler_t`
#### `PyOS_snprintf()`
#### `PyOS_string_to_double()`
#### `PyOS_strtol()`
#### `PyOS_strtoul()`
#### `PyOS_vsnprintf()`
#### `PyObject`
#### `PyObject.ob_refcnt`
#### `PyObject.ob_type`
#### `PyObject_ASCII()`
#### `PyObject_AsCharBuffer()`
#### `PyObject_AsFileDescriptor()`
#### `PyObject_AsReadBuffer()`
#### `PyObject_AsWriteBuffer()`
#### `PyObject_Bytes()`
#### `PyObject_Call()`
#### `PyObject_CallFunction()`
#### `PyObject_CallFunctionObjArgs()`
#### `PyObject_CallMethod()`
#### `PyObject_CallMethodObjArgs()`
#### `PyObject_CallNoArgs()`
#### `PyObject_CallObject()`
#### `PyObject_Calloc()`
#### `PyObject_CheckReadBuffer()`
#### `PyObject_ClearWeakRefs()`
#### `PyObject_DelItem()`
#### `PyObject_DelItemString()`
#### `PyObject_Dir()`
#### `PyObject_Format()`
#### `PyObject_Free()`
#### `PyObject_GC_Del()`
#### `PyObject_GC_IsFinalized()`
#### `PyObject_GC_IsTracked()`
#### `PyObject_GC_Track()`
#### `PyObject_GC_UnTrack()`
#### `PyObject_GenericGetAttr()`
#### `PyObject_GenericGetDict()`
#### `PyObject_GenericSetAttr()`
#### `PyObject_GenericSetDict()`
#### `PyObject_GetAIter()`
#### `PyObject_GetAttr()`
#### `PyObject_GetAttrString()`
#### `PyObject_GetItem()`
#### `PyObject_GetIter()`
#### `PyObject_HasAttr()`
#### `PyObject_HasAttrString()`
#### `PyObject_Hash()`
#### `PyObject_HashNotImplemented()`
#### `PyObject_Init()`
#### `PyObject_InitVar()`
#### `PyObject_IsInstance()`
#### `PyObject_IsSubclass()`
#### `PyObject_IsTrue()`
#### `PyObject_Length()`
#### `PyObject_Malloc()`
#### `PyObject_Not()`
#### `PyObject_Realloc()`
#### `PyObject_Repr()`
#### `PyObject_RichCompare()`
#### `PyObject_RichCompareBool()`
#### `PyObject_SelfIter()`
#### `PyObject_SetAttr()`
#### `PyObject_SetAttrString()`
#### `PyObject_SetItem()`
#### `PyObject_Size()`
#### `PyObject_Str()`
#### `PyObject_Type()`
#### `PyProperty_Type`
#### `PyRangeIter_Type`
#### `PyRange_Type`
#### `PyReversed_Type`
#### `PySeqIter_New()`
#### `PySeqIter_Type`
#### `PySequence_Check()`
#### `PySequence_Concat()`
#### `PySequence_Contains()`
#### `PySequence_Count()`
#### `PySequence_DelItem()`
#### `PySequence_DelSlice()`
#### `PySequence_Fast()`
#### `PySequence_GetItem()`
#### `PySequence_GetSlice()`
#### `PySequence_In()`
#### `PySequence_InPlaceConcat()`
#### `PySequence_InPlaceRepeat()`
#### `PySequence_Index()`
#### `PySequence_Length()`
#### `PySequence_List()`
#### `PySequence_Repeat()`
#### `PySequence_SetItem()`
#### `PySequence_SetSlice()`
#### `PySequence_Size()`
#### `PySequence_Tuple()`
#### `PySetIter_Type`
#### `PySet_Add()`
#### `PySet_Clear()`
#### `PySet_Contains()`
#### `PySet_Discard()`
#### `PySet_New()`
#### `PySet_Pop()`
#### `PySet_Size()`
#### `PySet_Type`
#### `PySlice_AdjustIndices()`
#### `PySlice_GetIndices()`
#### `PySlice_GetIndicesEx()`
#### `PySlice_New()`
#### `PySlice_Type`
#### `PySlice_Unpack()`
#### `PyState_AddModule()`
#### `PyState_FindModule()`
#### `PyState_RemoveModule()`
#### `PyStructSequence_Desc`
#### `PyStructSequence_Field`
#### `PyStructSequence_GetItem()`
#### `PyStructSequence_New()`
#### `PyStructSequence_NewType()`
#### `PyStructSequence_SetItem()`
#### `PySuper_Type`
#### `PySys_AddWarnOption()`
#### `PySys_AddWarnOptionUnicode()`
#### `PySys_AddXOption()`
#### `PySys_FormatStderr()`
#### `PySys_FormatStdout()`
#### `PySys_GetObject()`
#### `PySys_GetXOptions()`
#### `PySys_HasWarnOptions()`
#### `PySys_ResetWarnOptions()`
#### `PySys_SetArgv()`
#### `PySys_SetArgvEx()`
#### `PySys_SetObject()`
#### `PySys_SetPath()`
#### `PySys_WriteStderr()`
#### `PySys_WriteStdout()`
#### `PyThreadState`
#### `PyThreadState_Clear()`
#### `PyThreadState_Delete()`
#### `PyThreadState_Get()`
#### `PyThreadState_GetDict()`
#### `PyThreadState_GetFrame()`
#### `PyThreadState_GetID()`
#### `PyThreadState_GetInterpreter()`
#### `PyThreadState_New()`
#### `PyThreadState_SetAsyncExc()`
#### `PyThreadState_Swap()`
#### `PyThread_GetInfo()`
#### `PyThread_ReInitTLS()`
#### `PyThread_acquire_lock()`
#### `PyThread_acquire_lock_timed()`
#### `PyThread_allocate_lock()`
#### `PyThread_create_key()`
#### `PyThread_delete_key()`
#### `PyThread_delete_key_value()`
#### `PyThread_exit_thread()`
#### `PyThread_free_lock()`
#### `PyThread_get_key_value()`
#### `PyThread_get_stacksize()`
#### `PyThread_get_thread_ident()`
#### `PyThread_get_thread_native_id()`
#### `PyThread_init_thread()`
#### `PyThread_release_lock()`
#### `PyThread_set_key_value()`
#### `PyThread_set_stacksize()`
#### `PyThread_start_new_thread()`
#### `PyThread_tss_alloc()`
#### `PyThread_tss_create()`
#### `PyThread_tss_delete()`
#### `PyThread_tss_free()`
#### `PyThread_tss_get()`
#### `PyThread_tss_is_created()`
#### `PyThread_tss_set()`
#### `PyTraceBack_Here()`
#### `PyTraceBack_Print()`
#### `PyTraceBack_Type`
#### `PyTupleIter_Type`
#### `PyTuple_GetItem()`
#### `PyTuple_GetSlice()`
#### `PyTuple_New()`
#### `PyTuple_Pack()`
#### `PyTuple_SetItem()`
#### `PyTuple_Size()`
#### `PyTuple_Type`
#### `PyTypeObject`
#### `PyType_ClearCache()`
#### `PyType_FromModuleAndSpec()`
#### `PyType_FromSpec()`
#### `PyType_FromSpecWithBases()`
#### `PyType_GenericAlloc()`
#### `PyType_GenericNew()`
#### `PyType_GetFlags()`
#### `PyType_GetModule()`
#### `PyType_GetModuleState()`
#### `PyType_GetSlot()`
#### `PyType_IsSubtype()`
#### `PyType_Modified()`
#### `PyType_Ready()`
#### `PyType_Slot`
#### `PyType_Spec`
#### `PyType_Type`
#### `PyUnicodeDecodeError_Create()`
#### `PyUnicodeDecodeError_GetEncoding()`
#### `PyUnicodeDecodeError_GetEnd()`
#### `PyUnicodeDecodeError_GetObject()`
#### `PyUnicodeDecodeError_GetReason()`
#### `PyUnicodeDecodeError_GetStart()`
#### `PyUnicodeDecodeError_SetEnd()`
#### `PyUnicodeDecodeError_SetReason()`
#### `PyUnicodeDecodeError_SetStart()`
#### `PyUnicodeEncodeError_GetEncoding()`
#### `PyUnicodeEncodeError_GetEnd()`
#### `PyUnicodeEncodeError_GetObject()`
#### `PyUnicodeEncodeError_GetReason()`
#### `PyUnicodeEncodeError_GetStart()`
#### `PyUnicodeEncodeError_SetEnd()`
#### `PyUnicodeEncodeError_SetReason()`
#### `PyUnicodeEncodeError_SetStart()`
#### `PyUnicodeIter_Type`
#### `PyUnicodeTranslateError_GetEnd()`
#### `PyUnicodeTranslateError_GetObject()`
#### `PyUnicodeTranslateError_GetReason()`
#### `PyUnicodeTranslateError_GetStart()`
#### `PyUnicodeTranslateError_SetEnd()`
#### `PyUnicodeTranslateError_SetReason()`
#### `PyUnicodeTranslateError_SetStart()`
#### `PyUnicode_Append()`
#### `PyUnicode_AppendAndDel()`
#### `PyUnicode_AsASCIIString()`
#### `PyUnicode_AsCharmapString()`
#### `PyUnicode_AsDecodedObject()`
#### `PyUnicode_AsDecodedUnicode()`
#### `PyUnicode_AsEncodedObject()`
#### `PyUnicode_AsEncodedString()`
#### `PyUnicode_AsEncodedUnicode()`
#### `PyUnicode_AsLatin1String()`
#### `PyUnicode_AsMBCSString()`
#### `PyUnicode_AsRawUnicodeEscapeString()`
#### `PyUnicode_AsUCS4()`
#### `PyUnicode_AsUCS4Copy()`
#### `PyUnicode_AsUTF16String()`
#### `PyUnicode_AsUTF32String()`
#### `PyUnicode_AsUTF8AndSize()`
#### `PyUnicode_AsUTF8String()`
#### `PyUnicode_AsUnicodeEscapeString()`
#### `PyUnicode_AsWideChar()`
#### `PyUnicode_AsWideCharString()`
#### `PyUnicode_BuildEncodingMap()`
#### `PyUnicode_Compare()`
#### `PyUnicode_CompareWithASCIIString()`
#### `PyUnicode_Concat()`
#### `PyUnicode_Contains()`
#### `PyUnicode_Count()`
#### `PyUnicode_Decode()`
#### `PyUnicode_DecodeASCII()`
#### `PyUnicode_DecodeCharmap()`
#### `PyUnicode_DecodeCodePageStateful()`
#### `PyUnicode_DecodeFSDefault()`
#### `PyUnicode_DecodeFSDefaultAndSize()`
#### `PyUnicode_DecodeLatin1()`
#### `PyUnicode_DecodeLocale()`
#### `PyUnicode_DecodeLocaleAndSize()`
#### `PyUnicode_DecodeMBCS()`
#### `PyUnicode_DecodeMBCSStateful()`
#### `PyUnicode_DecodeRawUnicodeEscape()`
#### `PyUnicode_DecodeUTF16()`
#### `PyUnicode_DecodeUTF16Stateful()`
#### `PyUnicode_DecodeUTF32()`
#### `PyUnicode_DecodeUTF32Stateful()`
#### `PyUnicode_DecodeUTF7()`
#### `PyUnicode_DecodeUTF7Stateful()`
#### `PyUnicode_DecodeUTF8()`
#### `PyUnicode_DecodeUTF8Stateful()`
#### `PyUnicode_DecodeUnicodeEscape()`
#### `PyUnicode_EncodeCodePage()`
#### `PyUnicode_EncodeFSDefault()`
#### `PyUnicode_EncodeLocale()`
#### `PyUnicode_FSConverter()`
#### `PyUnicode_FSDecoder()`
#### `PyUnicode_Find()`
#### `PyUnicode_FindChar()`
#### `PyUnicode_Format()`
#### `PyUnicode_FromEncodedObject()`
#### `PyUnicode_FromFormat()`
#### `PyUnicode_FromFormatV()`
#### `PyUnicode_FromObject()`
#### `PyUnicode_FromOrdinal()`
#### `PyUnicode_FromString()`
#### `PyUnicode_FromStringAndSize()`
#### `PyUnicode_FromWideChar()`
#### `PyUnicode_GetDefaultEncoding()`
#### `PyUnicode_GetLength()`
#### `PyUnicode_GetSize()`
#### `PyUnicode_InternFromString()`
#### `PyUnicode_InternImmortal()`
#### `PyUnicode_InternInPlace()`
#### `PyUnicode_IsIdentifier()`
#### `PyUnicode_Join()`
#### `PyUnicode_Partition()`
#### `PyUnicode_RPartition()`
#### `PyUnicode_RSplit()`
#### `PyUnicode_ReadChar()`
#### `PyUnicode_Replace()`
#### `PyUnicode_Resize()`
#### `PyUnicode_RichCompare()`
#### `PyUnicode_Split()`
#### `PyUnicode_Splitlines()`
#### `PyUnicode_Substring()`
#### `PyUnicode_Tailmatch()`
#### `PyUnicode_Translate()`
#### `PyUnicode_Type`
#### `PyUnicode_WriteChar()`
#### `PyVarObject`
#### `PyVarObject.ob_base`
#### `PyVarObject.ob_size`
#### `PyWeakReference`
#### `PyWeakref_GetObject()`
#### `PyWeakref_NewProxy()`
#### `PyWeakref_NewRef()`
#### `PyWrapperDescr_Type`
#### `PyWrapper_New()`
#### `PyZip_Type`
#### `Py_AddPendingCall()`
#### `Py_AtExit()`
#### `Py_BEGIN_ALLOW_THREADS`
#### `Py_BLOCK_THREADS`
#### `Py_BuildValue()`
#### `Py_BytesMain()`
#### `Py_CompileString()`
#### `Py_DecRef()`
#### `Py_DecodeLocale()`
#### `Py_END_ALLOW_THREADS`
#### `Py_EncodeLocale()`
#### `Py_EndInterpreter()`
#### `Py_EnterRecursiveCall()`
#### `Py_Exit()`
#### `Py_FatalError()`
#### `Py_FileSystemDefaultEncodeErrors`
#### `Py_FileSystemDefaultEncoding`
#### `Py_Finalize()`
#### `Py_FinalizeEx()`
#### `Py_GenericAlias()`
#### `Py_GenericAliasType`
#### `Py_GetBuildInfo()`
#### `Py_GetCompiler()`
#### `Py_GetCopyright()`
#### `Py_GetExecPrefix()`
#### `Py_GetPath()`
#### `Py_GetPlatform()`
#### `Py_GetPrefix()`
#### `Py_GetProgramFullPath()`
#### `Py_GetProgramName()`
#### `Py_GetPythonHome()`
#### `Py_GetRecursionLimit()`
#### `Py_GetVersion()`
#### `Py_HasFileSystemDefaultEncoding`
#### `Py_IncRef()`
#### `Py_Initialize()`
#### `Py_InitializeEx()`
#### `Py_Is()`
#### `Py_IsFalse()`
#### `Py_IsInitialized()`
#### `Py_IsNone()`
#### `Py_IsTrue()`
#### `Py_LeaveRecursiveCall()`
#### `Py_Main()`
#### `Py_MakePendingCalls()`
#### `Py_NewInterpreter()`
#### `Py_NewRef()`
#### `Py_ReprEnter()`
#### `Py_ReprLeave()`
#### `Py_SetPath()`
#### `Py_SetProgramName()`
#### `Py_SetPythonHome()`
#### `Py_SetRecursionLimit()`
#### `Py_UCS4`
#### `Py_UNBLOCK_THREADS`
#### `Py_UTF8Mode`
#### `Py_VaBuildValue()`
#### `Py_XNewRef()`
#### `Py_intptr_t`
#### `Py_ssize_t`
#### `Py_uintptr_t`
#### `allocfunc`
#### `binaryfunc`
#### `descrgetfunc`
#### `descrsetfunc`
#### `destructor`
#### `getattrfunc`
#### `getattrofunc`
#### `getiterfunc`
#### `getter`
#### `hashfunc`
#### `initproc`
#### `inquiry`
#### `iternextfunc`
#### `lenfunc`
#### `newfunc`
#### `objobjargproc`
#### `objobjproc`
#### `reprfunc`
#### `richcmpfunc`
#### `setattrfunc`
#### `setattrofunc`
#### `setter`
#### `ssizeargfunc`
#### `ssizeobjargproc`
#### `ssizessizeargfunc`
#### `ssizessizeobjargproc`
#### `symtable`
#### `ternaryfunc`
#### `traverseproc`
#### `unaryfunc`
#### `visitproc`














