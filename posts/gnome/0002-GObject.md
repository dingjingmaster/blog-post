---
title: "GObject"
date: 2023-01-08T21:35:22+08:00
tags: ['gobject', 'glib', 'gnome', 'linux']
categories: ['gnome']
draft: false
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
    // object_type 的 GTypeClass，不存在或者是动态加载，则返回NULL
    // object_type 表示 类的 TypeID
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
    // G_UNLIKELY 提示编译器，表达式不太可能计算为真值
    // 检测 GObject 构造函数是否使用自定义的构造，
    // 默认构造是：
    // static GObject* g_object_constructor(GType type, guint n_construct_properties, GObjectConstructParam *construct_params)
    if G_UNLIKELY(CLASS_HAS_CUSTOM_CONSTRUCTOR(class)) {
        return g_object_new_with_custom_constructor (class, params, n_params);
    }

    GObject* object = (GObject*) g_type_create_instance (class->g_type_class.g_type);

    if (CLASS_HAS_CUSTOM_CONSTRUCTED (class)) {
        class->constructed (object);
    }

    return object;
}
```

```c
#define CLASS_HAS_CUSTOM_CONSTRUCTOR(class) \
    ((class)->constructor != g_object_constructor)
```

### 小结

1. gobject 是类的基类
2. gobject 中关于类的实现其实是在 GType 中。具体：
    - `GObject` --> `GTypeInstance` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <=> `GTypeClass*` <=> `GType`
    - `GObjectClass` --> `GTypeClass` &nbsp;&nbsp;<=> `GType`
3. gobject 实例化主要过程：
    1. `g_object_new(GType)`
        1. `g_object_new_with_properties(GType, ...)`，根据 `GType` 获取到 `GObjectClass*` 指针
        2. `g_object_new_with_properties(GType, ...)`，使用`GObjectClass*` 调用 `g_object_new_internal(GObjectClass*)`，根据 `GObjectClass*` 指针获取到 `GObject*` 指针
            1. 在 `g_object_new_internal(GObjectClass*)`中，调用 `(GObject *) g_type_create_instance (class->g_type_class.g_type);`，其中 class 表示`GObjectClass`

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
// static 信息注册位置：static_type_nodes_ht
GType g_type_register_static (GType parent_type, const gchar* type_name, const GTypeInfo* info, GTypeFlags flags)
{
    GType type = 0;
    // static void gobject_init (void)
    // 函数里调用：static_quark_type_flags = g_quark_from_static_string ("-g-type-private--GTypeFlags");
    g_assert_type_system_initialized();

    // check_type_name_I: 根据 name 从 static_type_nodes_ht 获取 GType
    // check_derivation_I: 根据 父类 GType 获取到 TypeNode，根据 TypeNode 获取到 GTypeFundamentalInfo 确定继承深度是否可推倒
    if (!check_type_name_I (type_name) || !check_derivation_I (parent_type, type_name))
        return 0;

    //
    if (info->class_finalize) {
        return 0;
    }

    TypeNode* node = NULL;
    TypeNode* pnode = lookup_type_node_I (parent_type);

    G_WRITE_LOCK(&type_rw_lock);
    
    // 父类引用 +1
    type_data_ref_Wm(pnode);

    // #define NODE_FUNDAMENTAL_TYPE(node) (node->supers[node->n_supers])
    // GTypeFundamentalInfo* == type_node_fundamental_info_I(TypeNode)
    if (check_type_info_I (pnode, NODE_FUNDAMENTAL_TYPE(pnode), type_name, info)) {
        // 执行 g_hash_table_insert (static_type_nodes_ht, (gpointer) g_quark_to_string (node->qname), (gpointer) type);
        node = type_node_new_W (pnode, type_name, NULL);
        type_add_flags_W (node, flags);

        // #define NODE_TYPE(node) (node->supers[0])
        type = NODE_TYPE(node); 
        type_data_make_W (node, info, check_value_table_I (type_name, info->value_table) ? info->value_table : NULL);
    }
    G_WRITE_UNLOCK (&type_rw_lock);

    return type;
}

```

check_type_info_I
```c
static gboolean check_type_info_I (TypeNode* pnode, GType ftype, const gchar *type_name, const GTypeInfo *info)
{
    GTypeFundamentalInfo *finfo = type_node_fundamental_info_I (lookup_type_node_I (ftype));
    gboolean is_interface = ftype == G_TYPE_INTERFACE;

    g_assert (ftype <= G_TYPE_FUNDAMENTAL_MAX && !(ftype & TYPE_ID_MASK));

    if (!(finfo->type_flags & G_TYPE_FLAG_INSTANTIATABLE) && (info->instance_size || info->n_preallocs || info->instance_init)) {
        if (pnode) {
            g_critical ("cannot instantiate '%s', derived from non-instantiatable parent type '%s'", type_name, NODE_NAME (pnode));
        }
        else {
            g_critical ("cannot instantiate '%s' as non-instantiatable fundamental", type_name);
        }
	
        return FALSE;
    }

    /* check class & interface members */
    if (!((finfo->type_flags & G_TYPE_FLAG_CLASSED) || is_interface)
            && (info->class_init || info->class_finalize || info->class_data || info->class_size || info->base_init || info->base_finalize)) {
        if (pnode)
            g_critical ("cannot create class for '%s', derived from non-classed parent type '%s'", type_name, NODE_NAME (pnode));
        else
            g_critical ("cannot create class for '%s' as non-classed fundamental", type_name);
      
        return FALSE;
    }

    /* check interface size */
    if (is_interface && info->class_size < sizeof (GTypeInterface)) {
        g_critical ("specified interface size for type '%s' is smaller than 'GTypeInterface' size", type_name);
        return FALSE;
    }

    /* check class size */
    if (finfo->type_flags & G_TYPE_FLAG_CLASSED) {
        if (info->class_size < sizeof (GTypeClass)) {
            g_critical ("specified class size for type '%s' is smaller than 'GTypeClass' size", type_name);
            return FALSE;
        }

        if (pnode && info->class_size < pnode->data->class.class_size) {
            g_critical ("specified class size for type '%s' is smaller than the parent type's '%s' class size", type_name, NODE_NAME (pnode));
	        return FALSE;
	    }
    }

    /* check instance size */
    if (finfo->type_flags & G_TYPE_FLAG_INSTANTIATABLE) {
        if (info->instance_size < sizeof (GTypeInstance)) {
            g_critical ("specified instance size for type '%s' is smaller than 'GTypeInstance' size", type_name);
	        return FALSE;
        }

        if (pnode && info->instance_size < pnode->data->instance.instance_size) {
            g_critical ("specified instance size for type '%s' is smaller than the parent type's '%s' instance size", type_name, NODE_NAME (pnode));
            return FALSE;
        }
    }

    return TRUE;
}
```

type_node_new_W

```c
static TypeNode* type_node_new_W (TypeNode* pnode, const gchar *name, GTypePlugin *plugin) 
{
    g_assert (pnode);
    g_assert (pnode->n_supers < MAX_N_SUPERS);
    g_assert (pnode->n_children < MAX_N_CHILDREN);
  
    return type_node_any_new_W (pnode, NODE_FUNDAMENTAL_TYPE (pnode), name, plugin, 0);
}
```

type_node_any_new_W

```c
static TypeNode* type_node_any_new_W (TypeNode* pnode, GType ftype, const gchar* name, GTypePlugin* plugin, GTypeFundamentalFlags type_flags)
{
    guint n_supers;
    GType type;
    TypeNode *node;
    guint i, node_size = 0;

    n_supers = pnode ? pnode->n_supers + 1 : 0;
  
    if (!pnode) {
        /* fundamental type info */
        node_size += SIZEOF_FUNDAMENTAL_INFO;
    }

    /* TypeNode structure */
    node_size += SIZEOF_BASE_TYPE_NODE ();	  

    /* self + ancestors + (0) for ->supers[] */              
    node_size += (sizeof (GType) * (1 + n_supers + 1));
    node = g_malloc0 (node_size);

    /* offset fundamental types */
    if (!pnode) {
        // #define SIZEOF_FUNDAMENTAL_INFO  ((gssize) MAX (MAX (sizeof (GTypeFundamentalInfo), sizeof (gpointer)), sizeof (glong)))
        // 去掉 GType 信息
        // TypeNode = GType + TypeNode
        // 
        node = G_STRUCT_MEMBER_P (node, SIZEOF_FUNDAMENTAL_INFO);
        static_fundamental_type_nodes[ftype >> G_TYPE_FUNDAMENTAL_SHIFT] = node;
        type = ftype;
    }
    else {
        type = (GType) node;
    }
  
    g_assert ((type & TYPE_ID_MASK) == 0);
    
    node->n_supers = n_supers;
    if (!pnode) {
        node->supers[0] = type;
        node->supers[1] = 0;
      
        node->is_classed = (type_flags & G_TYPE_FLAG_CLASSED) != 0;
        node->is_instantiatable = (type_flags & G_TYPE_FLAG_INSTANTIATABLE) != 0;
      
        if (NODE_IS_IFACE (node)) {
            IFACE_NODE_N_PREREQUISITES (node) = 0;
	        IFACE_NODE_PREREQUISITES (node) = NULL;
        }
        else {
	        _g_atomic_array_init (CLASSED_NODE_IFACES_ENTRIES (node));
        }
    }
    else {
        node->supers[0] = type;
        memcpy (node->supers + 1, pnode->supers, sizeof (GType) * (1 + pnode->n_supers + 1));
      
        node->is_classed = pnode->is_classed;
        node->is_instantiatable = pnode->is_instantiatable;
      
        if (NODE_IS_IFACE (node)) {
            IFACE_NODE_N_PREREQUISITES (node) = 0;
            IFACE_NODE_PREREQUISITES (node) = NULL;
        }
        else {
            guint j;
            IFaceEntries *entries;

            entries = _g_atomic_array_copy (CLASSED_NODE_IFACES_ENTRIES (pnode), IFACE_ENTRIES_HEADER_SIZE, 0);
            if (entries) {
                for (j = 0; j < IFACE_ENTRIES_N_ENTRIES (entries); j++) {
                    entries->entry[j].vtable = NULL;
                    entries->entry[j].init_state = UNINITIALIZED;
                }
                _g_atomic_array_update (CLASSED_NODE_IFACES_ENTRIES (node), entries);
            }
        }

        i = pnode->n_children++;
        pnode->children = g_renew (GType, pnode->children, pnode->n_children);
        pnode->children[i] = type;
    }

    TRACE(GOBJECT_TYPE_NEW(name, node->supers[1], type));

    node->plugin = plugin;
    node->n_children = 0;
    node->children = NULL;
    node->data = NULL;
    node->qname = g_quark_from_string (name);
    node->global_gdata = NULL;
  
    g_hash_table_insert (static_type_nodes_ht, (gpointer) g_quark_to_string (node->qname), (gpointer) type);

    g_atomic_int_inc ((gint *)&type_registration_serial);

    return node;
}
```

type_add_flags_W
```c
static void type_add_flags_W (TypeNode* node, GTypeFlags flags)
{
    guint dflags;
  
    g_return_if_fail ((flags & ~TYPE_FLAG_MASK) == 0);
    g_return_if_fail (node != NULL);
  
    if ((flags & TYPE_FLAG_MASK) && node->is_classed && node->data && node->data->class.class) {
        g_critical ("tagging type '%s' as abstract after class initialization", NODE_NAME (node));
    }
  
    dflags = GPOINTER_TO_UINT (type_get_qdata_L (node, static_quark_type_flags));
    dflags |= flags;
    
    type_set_qdata_W (node, static_quark_type_flags, GUINT_TO_POINTER (dflags));

    node->is_final = (flags & G_TYPE_FLAG_FINAL) != 0;
}
```

```c
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

    } 
    else if (node->is_classed) {
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
    } 
    else if (NODE_IS_IFACE (node)) {
        data = g_malloc0 (sizeof(IFaceData) + vtable_size);
        if (vtable_size)
            vtable = G_STRUCT_MEMBER_P (data, sizeof (IfaceData));
        data->iface.xxx = info->xxx;
        // ...
    } 
    else if (NODE_IS_BOXED(node)) {
        data = g_malloc0 (sizeof(BoxedData) + vtable_size);
        if (vtable_size)
            vtable = G_STRUCT_MEMBER_P (data, sizeof (BoxedData));
    } 
    else {
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

```c
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

```



### 接着GObject中代码继续追踪g\_type\_create\_instance

```c
GTypeInstance* g_type_create_instance (GType type)
{
    // 类型早已注册，后续追相关代码
    TypeNode* node = lookup_type_node_I (type);

    if (G_UNLIKELY (!node || !node->is_instantiatable)) {
        g_error ("cannot create new instance of invalid (non-instantiatable) type '%s'", type_descriptive_name_I (type));
    }

    /* G_TYPE_IS_ABSTRACT() is an external call: _U */
    if (G_UNLIKELY (!node->mutatable_check_cache && G_TYPE_IS_ABSTRACT (type))) {
        g_error ("cannot create instance of abstract (non-instantiatable) type '%s'", type_descriptive_name_I (type));
    }
    
    if (G_UNLIKELY (G_TYPE_IS_DEPRECATED (type))) {
        maybe_issue_deprecation_warning (type);
    }

    //
    GTypeClass* class = g_type_class_ref (type);

    gint private_size = node->data->instance.private_size;
    gint ivar_size = node->data->instance.instance_size;

    allocated = instance_alloc (private_size + ivar_size);

    GTypeInstance* instance = (GTypeInstance *) (allocated + private_size);

    for (i = node->n_supers; i > 0; i--) {
        TypeNode* pnode = lookup_type_node_I (node->supers[i]);
        if (pnode->data->instance.instance_init) {
            instance->g_class = pnode->data->instance.class;
            pnode->data->instance.instance_init (instance, class);
        }
    }

    instance->g_class = class;

    if (node->data->instance.instance_init)
        node->data->instance.instance_init (instance, class);
    
    TRACE(GOBJECT_OBJECT_NEW(instance, type));

    return instance;
}
```

lookup_type_node_I
```c
static inline TypeNode* lookup_type_node_I (GType utype)
{
    // #define	G_TYPE_FUNDAMENTAL_SHIFT	(2)
    // #define	G_TYPE_FUNDAMENTAL_MAX		(255 << G_TYPE_FUNDAMENTAL_SHIFT)                       // 1020
    // #define	TYPE_ID_MASK				((GType) ((1 << G_TYPE_FUNDAMENTAL_SHIFT) - 1))         // 3

    if (utype > G_TYPE_FUNDAMENTAL_MAX)
        return (TypeNode*) (utype & ~TYPE_ID_MASK);
    else
        return static_fundamental_type_nodes[utype >> G_TYPE_FUNDAMENTAL_SHIFT];
}
```

g_type_class_ref
```c
gpointer g_type_class_ref (GType type)
{
    gboolean holds_ref;
    TypeNode *node = lookup_type_node_I (type);

    // 引用 +1
    if (G_LIKELY (type_data_ref_U (node))) {
        if (G_LIKELY (g_atomic_int_get (&node->data->class.init_state) == INITIALIZED))
            return node->data->class.class;
        holds_ref = TRUE;
    }
    else {
        holds_ref = FALSE;
    }

    /* required locking order: 1) class_init_rec_mutex, 2) type_rw_lock */
    g_rec_mutex_lock (&class_init_rec_mutex); 

    /* we need an initialized parent class for initializing derived classes */
    // #define NODE_PARENT_TYPE(node)  (node->supers[1])
    // 递归初始化GObject的所有父类 GTypeClass，引用+1
    GType ptype = NODE_PARENT_TYPE (node);
    GTypeClass *pclass = ptype ? g_type_class_ref (ptype) : NULL;
    
    G_WRITE_LOCK (&type_rw_lock);

    if (!holds_ref) {
        // 递归初始化父类 GTypeInfo
        type_data_ref_Wm (node);
    }

    /* class uninitialized */
    if (!node->data->class.class) {
        // 递归构造 GTypeClass 类型，分配内存
        // 把对象及其父类构造函数放入到一个单链表，依次调用
        type_class_init_Wm (node, pclass);
    }

    G_WRITE_UNLOCK (&type_rw_lock);

    if (pclass) {
        g_type_class_unref (pclass);
    }

    g_rec_mutex_unlock (&class_init_rec_mutex);

    return node->data->class.class;
}
```

type_data_ref_U
```c
static inline gboolean type_data_ref_U (TypeNode *node)
{
    guint current;

    do {
        // #define NODE_REFCOUNT(node)  ((guint) g_atomic_int_get ((int*) &(node)->ref_count))
        current = NODE_REFCOUNT (node);
        if (current < 1)
            return FALSE;
        // 等价于：
        // if (*ref_count == current) { *ref_count = current + 1; return TRUE; } else return FALSE; }
    } while (!g_atomic_int_compare_and_exchange ((int *) &node->ref_count, current, current + 1));

    return TRUE;
}
```

### 结论(针对GObject无参构造类)

1. 类的实例由 `struct _GObject` 和 `struct _GObjectClass` 两个结构组成，其中 `struct _GObject` 继承自 `GTypeInstance` 并增加了引用计数次数和GData数据；`struct _GObjectClass` 继承自 `GTypeClass` 并增加了以下几类信息：构造参数列表、构造和析构函数、属性的set/get。
2. 无论是 `GTypeInstance` 还是 `GTypeClass` 都继承自 `GType`，创建GObject的过程就是创建 `GType` 并调用 `构造函数` 的过程，大致过程跟踪 `g_object_new` 这一函数可知，但是详细过程都在`GType`里。
3. g_type_create_instance
    1. g_type_class_ref(GType) 这里会递归为父类分配空间，并依次调用父类构造函数
    2. 调用自己类的构造函数

> 问题：如何根据 GType 获取到父类？

```c
#define NODE_PARENT_TYPE(node) (node->supers[1])

// 注册的时候 获取所有父类保存到 node->supers 中，node->supers[0]是自己，node->supers[1]是父类，不支持多继承
```

## 写一个类

例子很多了：[https://github.com/dingjingmaster/demo/tree/master/gobject](https://github.com/dingjingmaster/demo/tree/master/gobject)

其中最简单的例子：[demo1](https://github.com/dingjingmaster/demo/tree/master/gobject/demo1)中有两个关键的宏：
- `#define DEMO_TYPE_DLIST (demo_dlist_get_type())`、`GType demo_dlist_get_type (void);`
- G_DEFINE_TYPE (demo_list_t, demo_dlist, G_TYPE_OBJECT);

> 主要看 `G_DEFINE_TYPE`， 定义在 `gtype.h` 中

G_DEFINE_TYPE
```c
// TN   表示类名，大小写字幕+下划线组成，一般使用驼峰命名
// t_n  表示类名，大小写字幕+下划线组成，一般使用大小写+下划线命名
// T_P  表示父类的 GType，比如：G_TYPE_OBJECT
#define G_DEFINE_TYPE(TN, t_n, T_P)         G_DEFINE_TYPE_EXTENDED (TN, t_n, T_P, 0, {})
```

G_DEFINE_TYPE_EXTENDED
```c
// TN   类名
// t_n  类名
// T_P  父类 GType
// _f_  GTypeFlags to pass to g_type_register_static()
// _C_  Custom code that gets inserted in the `*_get_type()` function。这个语句会执行
//
// 例子：
// G_DEFINE_TYPE_EXTENDED (GtkGadget, gtk_gadget, GTK_TYPE_WIDGET, 0, \
//      G_ADD_PRIVATE (GtkGadget) G_IMPLEMENT_INTERFACE (TYPE_GIZMO, gtk_gadget_gizmo_init));
//
// _G_DEFINE_TYPE_EXTENDED_BEGIN:
//  1. 在 _G_DEFINE_TYPE_EXTENDED_BEGIN_PRE 中定义了一系列函数
//  2. 在 _G_DEFINE_TYPE_EXTENDED_BEGIN_REGISTER 中主要调用：g_type_register_static_simple
//  3. 调用用户自定义函数：_C_
//
// _G_DEFINE_TYPE_EXTENDED_END 结束
// 
#define G_DEFINE_TYPE_EXTENDED(TN, t_n, T_P, _f_, _C_)  \
    _G_DEFINE_TYPE_EXTENDED_BEGIN (TN, t_n, T_P, _f_) {_C_;} \
    _G_DEFINE_TYPE_EXTENDED_END()
```

_G_DEFINE_TYPE_EXTENDED_BEGIN
```c
// 在 _G_DEFINE_TYPE_EXTENDED_BEGIN_PRE 中定义了一系列函数
// 在 _G_DEFINE_TYPE_EXTENDED_BEGIN_REGISTER 中主要调用：g_type_register_static_simple
#define _G_DEFINE_TYPE_EXTENDED_BEGIN(TypeName, type_name, TYPE_PARENT, flags) \
    _G_DEFINE_TYPE_EXTENDED_BEGIN_PRE(TypeName, type_name, TYPE_PARENT) \
    _G_DEFINE_TYPE_EXTENDED_BEGIN_REGISTER(TypeName, type_name, TYPE_PARENT, flags) \
```

_G_DEFINE_TYPE_EXTENDED_BEGIN_PRE
```c
#define _G_DEFINE_TYPE_EXTENDED_BEGIN_PRE(TypeName, type_name, TYPE_PARENT) \
\
static void     type_name##_init              (TypeName        *self); \
static void     type_name##_class_init        (TypeName##Class *klass); \
static GType    type_name##_get_type_once     (void); \
static gpointer type_name##_parent_class = NULL; \
static gint     TypeName##_private_offset; \
\
_G_DEFINE_TYPE_EXTENDED_CLASS_INIT(TypeName, type_name) \
\
/* 获取类的私有成员结构体 */\
G_GNUC_UNUSED static inline gpointer type_name##_get_instance_private (TypeName *self) \
{ \
  return (G_STRUCT_MEMBER_P (self, TypeName##_private_offset)); \
} \
\
/* 实现： */\
GType type_name##_get_type (void) \
{ \
  static gsize static_g_define_type_id = 0;
  /* Prelude goes here */

/* Added for _G_DEFINE_TYPE_EXTENDED_WITH_PRELUDE */
#define _G_DEFINE_TYPE_EXTENDED_BEGIN_REGISTER(TypeName, type_name, TYPE_PARENT, flags) \
  if (g_once_init_enter (&static_g_define_type_id)) \
    { \
      GType g_define_type_id = type_name##_get_type_once (); \
      g_once_init_leave (&static_g_define_type_id, g_define_type_id); \
    }					\
  return static_g_define_type_id; \
} /* closes type_name##_get_type() */ \
\
\
G_NO_INLINE static GType type_name##_get_type_once (void) \
{ \
  GType g_define_type_id = \
        /**/ \
        g_type_register_static_simple (TYPE_PARENT, \
                                       g_intern_static_string (#TypeName), /*gquark.c里有定义，确保字符串被gquark计算后的值是唯一的*/\
                                       sizeof (TypeName##Class), \
                                       (GClassInitFunc)(void (*)(void)) type_name##_class_intern_init, \
                                       sizeof (TypeName), \
                                       (GInstanceInitFunc)(void (*)(void)) type_name##_init, \
                                       (GTypeFlags) flags); \
    { /* custom code follows */
#define _G_DEFINE_TYPE_EXTENDED_END()	\
      /* following custom code */	\
    }					\
  return g_define_type_id; \
} /* closes type_name##_get_type_once() */
```

```c
#define _G_DEFINE_TYPE_EXTENDED_CLASS_INIT(TypeName, type_name) \
static void     type_name##_class_intern_init (gpointer klass) \
{ \
  type_name##_parent_class = g_type_class_peek_parent (klass); \
  type_name##_class_init ((TypeName##Class*) klass); \
}
```

```c
GType g_type_register_static_simple (GType parent_type, const gchar* type_name, guint class_size, GClassInitFunc class_init, guint instance_size, GInstanceInitFunc instance_init, GTypeFlags flags)
{
    GTypeInfo info;

    /* Instances are not allowed to be larger than this. If you have a big
    * fixed-length array or something, point to it instead.
    */
    g_return_val_if_fail (class_size <= G_MAXUINT16, G_TYPE_INVALID);
    g_return_val_if_fail (instance_size <= G_MAXUINT16, G_TYPE_INVALID);

    info.class_size = class_size;
    info.base_init = NULL;
    info.base_finalize = NULL;
    info.class_init = class_init;
    info.class_finalize = NULL;
    info.class_data = NULL;
    info.instance_size = instance_size;
    info.n_preallocs = 0;
    info.instance_init = instance_init;
    info.value_table = NULL;

    return g_type_register_static (parent_type, type_name, &info, flags);
}
```

至此，GObject创建过程走完了...

## 总结

GObject 类创建过程(G_DEFINE_TYPE)：
1. 声明一系列静态变量和函数
2. 定义`type_name##_get_type()`为了获取到GType; 其中调用了 `g_type_register_static_simple` 将类型注册到 GObject 中（一个全局静态的hashmap，其中GType通过）
3. 主调函数里使用 `g_object_new(type_name##_get_type, NULL)`;得到GObject，其中 GType 是 类对应 TypeNode 首地址转为 unsigned long 得到