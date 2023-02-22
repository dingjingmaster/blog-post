---
title: "ncurses使用中需要明确的点"
date: 2023-02-05T21:44:56+08:00
tags: ['ncurses']
categories: ['ncurses', 'linux']
draft: false
---

### ncurses 与 curses

ncurses 完全兼容旧的 curses，它们表示同一个函数库

### initscr()

初始化终端，清空屏幕，为 **当前窗口(stdscr)** 和其它数据结构分配内存。

### refresh()

对`stdscr`任何操作都是对屏幕缓存的操作，只有执行 `refresh()` 之后才把绘制结果显示到屏幕。

### endwin()

释放 curses 子系统及其数据结构占用的内存，并将终端置于正常模式。不调用并结束进程会导致终端显示异常。

### raw()和cbreak()

执行后，用户的每个输入都会被传递给程序，否则直到用户输入回车才传递给程序。

### echo()和noecho()

`noecho()` 用于控制回显，不执行则用户每次输入会立刻显示到当前(x，y)座标

### keypad()

用于初始化读取功能键

### halfdelay()

等待用户输入过程时可以超时返回。

### getch()

用于获取用户输入的一个字符

