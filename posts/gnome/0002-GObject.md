---
title: "GObject"
date: 2023-01-08T21:35:22+08:00
tags: ['gobject', 'glib', 'gnome', 'linux']
categories: ['gnome']
draft: true
---

## GObject

### 数据结构

> `GObject` 是基对象类型，在`GObject`结构体中的所有字段都是私有的，无法直接访问。<br/>
> 从 GLib2.72开始，GObject都至少与最大的基本GLib类型对齐(通常是guint64或者gdouble)。这条规则对`GObject`及其派生类或者由`G_ADD_PRIVATE()`添加的私有类都适用（然而栈上结构体大小不能超过64KiB，因此需要如果所写类所占空间太大则应该使用堆**heep**上存储）。

**主要**以下是 `struct _GObject` 的定义，具体定义参看`<root>/gobject/gobject.h` *(其中\<root\>表示glib源码根目录)*
```c
struct _GObject
{
    GTypeInstance   g_type_instance;    // gsize(unsigned long) 用以表明类型

    /* private */
    guint           ref_count;          // atomic，引用计数
    GData*          qdata;
};
```

**主要**以下是`struct _GObjectConstructParam` 的定义
```c
struct _GObjectConstructParam
{
    GParamSpec*     pspec;              // 构造函数的名
    GValue*         value;              // 构造函数值
};
```

**主要**以下是 `struct _GObjectClass` 的定义：
```c
struct _GObjectClass
{
    GTypeClass          g_type_class;
    
    /* private */
    GSList*             construct_properties;

    /* public */
    /* 很少重写 */
    GObject*    (*constructor)  (GType type, guint n_construct_properties, GObjectConstructParam* construct_properties);

    /* 可重写的方法 */
    void        (*set_property) (GObject* object, guint property_id, const GValue* value, GParamSpec* pspec);
    void        (*get_property) (GObject* object, guint property_id, GValue* value, GParamSpec* pspec);

    void        (*dispose)      (GObject* object);
    void        (*finalize)     (GObject* object);

    /* 很少重写 */
    void        (*dispatch_properties_changed)  (GObject* object, guint n_pspecs, GParamSpec** pspec);

    /* 信号 */
    void        (notify)        (GObject* object, GParamSpec* pspec);

    /* 当 constructor 执行完成之后调用 */
    void        (*constructed)  (GObject* object);

    /* private */
    gsize               flags;
    gsize               n_construct_properties;

    gpointer            pspecs;
    gsize               n_pspecs;

    /* padding */
    gpointer            pdummy[3];
};
```

其它结构定义:
1. `struct _GData`
```c
struct _GData
{
    guint32             len;        // 元素个数
    guint32             alloc;      // 已分配元素的数量
    GDataElt            data[1];    //
};
```

2. `struct _GDataElt`
```c
struct _GDataElt
{
    GQuark              key;        // 将字符串映射到唯一整数(hash)
    gpointer            data;
    GDestroyNotify      destroy;    // typedef void (*GDestroyNotify) (gpointer data);
};
```

3. `struct _GTypeInstance`
```c
struct _GTypeInstance
{
    /* private */
    GTypeClass*         g_class;
};
```

4. `struct _GTypeClass`
```c
struct _GTypeClass
{
    /* private */
    GType               g_type;     // gsize
};
```
### g\_object\_new

主要流程（假设没有参数传入，即：无参构造）
```c
gpointer g_object_new (GType object_type, const gchar* first_property_name, ...)
{
    return g_object_new_with_properties (object_type, 0, NULL, NULL);
}

```

```c
GObject* g_object_new_with_properties (GType object_type, guint n_properties, const char* names[], const GValue value[])
{
    GObjectClass* class = g_type_class_peek_static (object_type);

    if (NULL == class) {
        class = g_type_class_ref (object_type);
    }

    GObject* obj = g_object_new_internal (class, NULL, 0);

    return obj;
}
```

```c
static gpointer g_object_new_internal (GObjectClass* class, GObjectConstructParam* param, guint n_params)
{
    if (CLASS_HAS_CUSTOM_CONSTRUCTOR(class)) {
        return g_object_new with_custom_constructor (class, params, n_params);
    }

    GObject* object = (GObject*) g_type_create_instance (class->g_type_class.g_type);

    if (CLASS_HAS_CUSTOM_CONSTRUCTED (class)) {
        class->constructed (object);
    }

    return object;
}
```

### 小结

1. gobject 是类的基类
2. gobject 中关于类的实现其实实在 GType 中。具体：
    - `GObject` --> `GTypeInstance` <=> `GTypeClass*` <=> `GType`
    - `GObjectClass` --> `GTypeClass` <=> `GType`

## GType

> `GType` 是 `GObject` 系统的基石。它提供了`注册(registering)`和`管理(managing)`所有基本数据类型、用户自定义对象、接口类型的工具。
> <br/>

### GType 创建
GType类型的`创建`和`注册`共有两种方式：`静态类型`和`动态类型`。

1. `静态类型`永远不会像动态类型那样在运行时候加载或卸载，静态类型通过`g_type_register_static()`创建，创建后可通过`GTypeInfo`结构体来获取一些特殊信息
2. `动态类型`是使用`g_type_register_dynamic()`创建的，创建后信息保存在`GTypePlugin`结构中，具体可以使用`g_type_plugin_*()`接口获取。

> 注册类型之前已经默认调用了 `gobject_init`，在`g_object_init`里自动调用`glib_init`.

<br/>

GType 提供的注册函数在整个程序声明周期内只会调用一次，这类注册函数唯一的目的就是返回指定类的类型标识符。一旦注册成功这一类型（类或者接口），就可以实例化、继承或者实现它。

<br/>

另外还有一类注册函数用于注册基本类型，名为`g_type_register_fundamental()`，它同时需要`GTypeInfo`结构体和`GTypeFundmentalInfo`结构体完成注册，但是很少使用，因为大多数基本类型都是预定义的，而不是用户定义的。

<br/>

类型实例和类的结构体大小被限制最大为 64KiB（包括父类），类型实例的私有数据（由G\_ADD\_PRIVATED()创建）被限制为总共64KiB。

<br/>

GType类型名必须至少三个及以上字符组成，组成字符需满足C语言变量命名规则（数字字母下划线，数字不开头）。

### 数据结构

```c
typedef gsize GType;

struct _GTypeClass
{
    GType           g_type;
};

struct _GTypeInstance
{
    GTypeClass*     g_class;
};

struct _GTypeInterface
{
    GType           g_type;
    GType           g_instance_type;
};
```

```c
struct _GTypeInfo
{
    /*  */
    guint16             class_size;
    GBaseInitFunc       base_init;
    GBaseFinalizeFunc   base_finalize;

    /*  */
    GClassInitFunc      class_init;
    GClassFinalizeFunc  class_finalize;
    gconstpointer       class_data;

    /*  */
    guint16             instance_size;
    guint16             n_preallocs;
    GInstanceInitFunc   instance_init;

    /* value handling */
    const GTypeValueTable* value_table;
};
```

```c
struct _GTypeFundamentalInfo
{
    GTypeFundamentalFlags       type_flags;
};
```

```c
struct _GInterfaceInfo
{
    GInterfaceInitFunc          interface_init;
    GInterfaceFinalizeFunc      interface_finalize;
    gpointer                    interface_data;
};
```

```c
struct _GTypeValueTable
{
    void    (*value_init)   (GValue* value);
    void    (*value_free)   (GValue* value);
    void    (*value_copy)   (const GValue* src_value, GValue* dest_value);

    /**/
    gpointer (*value_peek_pointer)   (const GValue* value);
    const gchar* collect_value;
    gchar*  (*collect_value)    (GValue* value, guint n_collect_values, GTypeCValue* collect_values, guint collect_flags);

    const gchar* lcopy_format;
    gchar*  (*lcopy_value)  (const GValue* value, guint n_collect_values, GTypeCValue* collect_values, guint collect_flags);
};
```

很重要的两个全局变量:
```c
static GHashTable*      static_type_nodes_ht = NULL;
static TypeNode*        static_fundamental_type_nodes[(G_TYPE_FUNDAMENTAL_MAX >> G_TYPE_FUNDAMENTAL_SHITF) + 1] = {NULL, };
```

### g\_type\_register\_static

```c
GType g_type_register_static (GType parent_type, const gchar* type_name, const GTypeInfo* info, GTypeFlags flags)
{
    GType type = 0;
    g_type_system_initialized();

    if (!check_type_name_I (type_name) || !check_derivation_I (parent_type, type_name))
        return 0;

    if (info->class_finalize) {
        return 0;
    }

    TypeNode* node = NULL;
    TypeNode* pnode = lookup_type_node_I (parent_type);
    G_WRITE_LOCK(&type_rw_lock);
    type_data_ref_Wm(pnode);
    if (check_type_info_I (pnode, NODE_FUNDAMENTAL_TYPE(pnode), type_name, info)) {
        node = type_node_new_W (pnode, type_name, NULL);
        type_add_flags_W (node, flags);

        // ...这里
        type = NODE_TYPE(node);
        type_data_make_W (node, info, check_value_table_I (type_name, info->value_table) ? info->value_table : NULL);
    }
    G_WRITE_UNLOCK (&type_rw_lock);

    return type;
}

// ...
static TypeNode* type_node_new_W (TypeNode* pnode, const gchar* name, GTypePlugin* plugin)
{
    g_assert (pnode && (pnode->n_suppers < MAX_N_SUPERS) && (pnode->n_children < MAX_N_CHILDREN));

    return type_node_any_new_W (pnode, NODE_FUNDAMENTAL_TYPE(pnode), name, plugin, 0);
}

// ...
static TypeNode* type_node_any_new_W (TypeNode* pnode, GType ftype, const gchar* name, GTypePlugin* plugin, GTypeFundamentalFlags type_flags)
{
    guint n_supers = pnode ? pnode->n_supers + 1 : 0;

    if (!pnode) {
        node_size += SIZEOF_FUNDAMENTAL_INFO;
    }

    node_size += SIZEOF_BASE_TYPE_NODE();
    node_size += (sizeof (GType) * (1 + n_supers + 1));

    TypeNode* node = g_malloc0 (node_size);

    GType type = (GType) node;

    g_assert ((type & TYPE_ID_MASK) == 0);

    node->n_supers = n_supers;

    node->supers[0] = type;
    memcpy (node->supers + 1, pnode->supers, sizeof (GType) * (1 + pnode->n_supers + 1));

    node->is_classed = node->is_classed;
    node->is_instantiatable = pnode->is_instantiatable;

    IFaceEntries* entries = _g_atomic_array_copy (CLASSED_NODE_IFACES_ENTRIES(pnode), IFACE_ENTRIES_HEADER_SIZE, 0);
    if (entries) {
        for (int j = 0; j < IFACE_ENTRIES_N_ENTRIES(entries); ++j) {
            entries->entry[j].vtable = NULL;
            entries->entry[j].init_state = UNINITIALIZED;
        }
        _g_atomic_array_update (CLASSED_NODE_IFACES_ENTRIES(node), entries);
    }

    guint i = pnode->n_children++;
    pnode->children = g_renew (GType, pnode->children, pnode->n_children);
    pnode->children[i] = type;

    GOBJECT_TYPE_NEW (name, node->supers[1], type);

    node->plugin = NULL;
    node->c_children = 0;
    node->children = NULL;
    node->data = NULL;
    node->qname = g_quark_from_string (name);
    node->global_gdata = NULL;
    g_hash_table_insert (static_type_nodes_ht, g_quark_to_string(node->qname), type);

    g_atomic_int_inc ((gint*) &type_registration_serial);

    return node;
}

static void type_add_flags_W (TypeNode* node, GTypeFlags flags)
{
    guint dflags = GPOINTER_TO_UINT (type_get_qdata_L (node, static_quark_type_flags));
    dflags |= flags;

    type_set_qdata_W (node, static_quark_type_flags, GUINT_TO_POINTER(dflags));

    node->is_final = (flags & G_TYPE_FLAG_FINAL) != 0;
}

static void type_data_make_W (TypeNode* node, const GTypeInfo* info, const GTypeValueTable* value_table)
{
    guint vtable_size = 0;
    TypeNode* pnode = lookup_type_node_I (NODE_PARENT_TYPE(node));
    if (pnode) {
        vtable = pnode->data->common.value_table;
    }

    if (value_table) {
        // 计算 vtable_size
    }

    if (node->is_instantiatable) {
        // is_instantiatable is also is_classed
        TypeNode* pnode = lookup_type_node_I (NODE_PARENT_TYPE(node));

        date = g_malloc0 (sizeof (InstanceData) + vtable_size);
        if (vtable_size)
            vtable = G_STRUCT_MEMBER_P (data, sizeof (InstanceData));
        data->instance.xxx = info->xxx;
        // ....


        data->instance.private_size = 0;
        data->instance.class_private_size = 0;
        if (pnode) {
            data->instance.class_private_size = pnode->data->instance.class_private_dize;
        }
        data->instance.n_preallocs = MIN (info->n_preallocs, 1024);
        data->instance.instance_init = info->instance_init;

    } else if (node->is_classed) {
        // only classed
        TypeNode* pnode = lookup_type_node_I (NODE_PARENT_TYPE(node));

        data = g_malloc0 (sizeof (ClassData) + vtable_size);
        if (vtable_size)
            vtable = G_STRUCT_MEMBER_P (data, sizeof (ClassData));
        data->class.xxx = info->xxx;
        // ...

        if (pnode)
            data->class.class_private_size = pnode->data->class.class_private_size;
        data->class.init_state = UNINITIALIZED;
    } else if (NODE_IS_IFACE (node)) {
        data = g_malloc0 (sizeof(IFaceData) + vtable_size);
        if (vtable_size)
            vtable = G_STRUCT_MEMBER_P (data, sizeof (IfaceData));
        data->iface.xxx = info->xxx;
        // ...
    } else if (NODE_IS_BOXED(node)) {
        data = g_malloc0 (sizeof(BoxedData) + vtable_size);
        if (vtable_size)
            vtable = G_STRUCT_MEMBER_P (data, sizeof (BoxedData));
    } else {
        data = g_malloc0 (sizeof(CommonData) + vtable_size);
        if (vtable_size)
            vtable = G_STRUCT_MEMBER_P (data, sizeof (CommonData));
    }

    node->data = data;

    if (vtable_size) {
        // ...
    }

    node->data->common.value_table = vtable;
    node->mutatable_check_cache = (node->data->common.value_table->value_init != NULL && !((G_TYPE_FLAG_VALUE_ABSTRACT | G_TYPE_FLAG_ABSTRACT) & G_POINTER_TO_UINT(type_get_qdata_L(node, static_quark_type_flags))));

    g_assert (node->data->common.value != NULL);

    g_atomic_int_set ((int*) &node->ref_count, 1);
}
```

## 结论

### 针对GObject无参构造类

1. 类的实例由 `struct _GObject` 和 `struct _GObjectClass` 两个结构组成，其中 `struct _GObject` 继承自 `GTypeInstance` 并增加了引用计数次数和GData数据；`struct _GObjectClass` 继承自 `GTypeClass` 并增加了以下几类信息：构造参数列表、构造和析构函数、属性的set/get。
2. 无论是 `GTypeInstance` 还是 `GTypeClass` 都继承自 `GType`，创建GObject的过程就是创建 `GType` 并调用 `构造函数` 的过程，大致过程跟踪 `g_object_new` 这一函数可知，但是详细过程都在`GType`里。


