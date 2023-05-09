---
title: "GObject使用——常见宏"
date: 2023-04-16T10:47:09+08:00
tags: ['linux', 'gnome', 'gobject']
categories: ['gnome']
draft: false
---

## GObject 常用宏

### `G_TYPE_IS_OBJECT(type)`

- 说明：检查传入的类型`type`是否为`G_TYPE_OBJECT`或其派生
```c
#define G_TYPE_IS_OBJECT(type)      (G_TYPE_FUNDAMENTAL (type) == G_TYPE_OBJECT)
```

### `G_OBJECT(obj)`

- 说明：将GObject或派生GObject的子类转为GObject

```c
#define G_OBJECT(object)            (G_TYPE_CHECK_INSTANCE_CAST ((object), G_TYPE_OBJECT, GObject))
```

### `G_OBJECT_CLASS(klass)`

- 说明：将GObjectClass或派生GObject的子类转为GObjectClass

```c
#define G_OBJECT_CLASS(class)       (G_TYPE_CHECK_CLASS_CAST ((class), G_TYPE_OBJECT, GObjectClass))
```

### `G_IS_OBJECT(object)`

- 说明：检查是否是 `G_TYPE_OBJECT` 类型

```c
#if GLIB_VERSION_MAX_ALLOWED >= GLIB_VERSION_2_42
#define G_IS_OBJECT(object)         (G_TYPE_CHECK_INSTANCE_FUNDAMENTAL_TYPE ((object), G_TYPE_OBJECT))
#else
#define G_IS_OBJECT(object)         (G_TYPE_CHECK_INSTANCE_TYPE ((object), G_TYPE_OBJECT))
#endif
```

### `G_IS_OBJECT_CLASS(klass)`

- 说明：检查`klass`是否是可用的GObjectClass结构

```c
#define G_IS_OBJECT_CLASS(class)    (G_TYPE_CHECK_CLASS_TYPE ((class), G_TYPE_OBJECT))
```

### `G_OBJECT_GET_CLASS(obj)`

- 说明：通过 GObject 实例得到 GObjectClass 实例

```c
#define G_OBJECT_GET_CLASS(object)  (G_TYPE_INSTANCE_GET_CLASS ((object), G_TYPE_OBJECT, GObjectClass))
```

### `G_OBJECT_TYPE(obj)`

- 说明：返回GObject对象的ID（`G_TYPE_OBJECT`对应的那个数字）

```c
#define G_OBJECT_TYPE(object)       (G_TYPE_FROM_INSTANCE (object))
```

### `G_OBJECT_TYPE_NAME(obj)`

- 说明：返回GObject对象的字符串(此字符串是对象注册时候用到的那个)

```c
#define G_OBJECT_TYPE_NAME(object)  (g_type_name (G_OBJECT_TYPE (object)))
```

### `G_OBJECT_CLASS_TYPE(klass)`

- 说明：根据GObjectClass 获取到 Type ID

```c
#define G_OBJECT_CLASS_TYPE(class)  (G_TYPE_FROM_CLASS (class))
```

### `G_OBJECT_CLASS_NAME(class)`

- 说明：根据类获取到GObject名字

```c
#define G_OBJECT_CLASS_NAME(class)  (g_type_name (G_OBJECT_CLASS_TYPE (class)))
```

### `G_VALUE_HOLDS_OBJECT(value)`

- 说明：检查给定 value 是否可以保存来自 `G_TYPE_OBJECT` 的值

```c
#define G_VALUE_HOLDS_OBJECT(value) (G_TYPE_CHECK_VALUE_TYPE ((value), G_TYPE_OBJECT))
```

### `G_TYPE_INITIALLY_UNOWNED`

- 说明：

```c
#define G_TYPE_INITIALLY_UNOWNED	            (g_initially_unowned_get_type())
#define G_INITIALLY_UNOWNED(object)             (G_TYPE_CHECK_INSTANCE_CAST ((object), G_TYPE_INITIALLY_UNOWNED, GInitiallyUnowned))
#define G_INITIALLY_UNOWNED_CLASS(class)        (G_TYPE_CHECK_CLASS_CAST ((class), G_TYPE_INITIALLY_UNOWNED, GInitiallyUnownedClass))
#define G_IS_INITIALLY_UNOWNED(object)          (G_TYPE_CHECK_INSTANCE_TYPE ((object), G_TYPE_INITIALLY_UNOWNED))
#define G_IS_INITIALLY_UNOWNED_CLASS(class)     (G_TYPE_CHECK_CLASS_TYPE ((class), G_TYPE_INITIALLY_UNOWNED))
#define G_INITIALLY_UNOWNED_GET_CLASS(object)   (G_TYPE_INSTANCE_GET_CLASS ((object), G_TYPE_INITIALLY_UNOWNED, GInitiallyUnownedClass))
```

> 我发现 IBusObject 中用到，暂时不明白功能，推测这个地方使用“原型设计模式”来创建其它类型

## GObject结构层次

```
    GObject
    ├── GBinding
    ├── GBindingGroup
    ├── GInitiallyUnowned
    ├── GSignalGroup
    ╰── GTypeModule
```

```c
typedef struct _GObject                  GObject;
typedef struct _GObjectClass             GObjectClass;

typedef struct _GObject                  GInitiallyUnowned;
typedef struct _GObjectClass             GInitiallyUnownedClass;
typedef struct _GObjectConstructParam    GObjectConstructParam;
```

