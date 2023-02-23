---
title: "ncurses API"
date: 2023-02-06T22:39:18+08:00
tags: ['ncurses']
categories: ['ncurses', 'linux']
draft: false
---

## add_wch

在窗口中添加**复杂字符串**，然后移动光标

```c
#include <curses.h>

int add_wch( const cchar_t *wch );
int wadd_wch( WINDOW *win, const cchar_t *wch );
int mvadd_wch( int y, int x, const cchar_t *wch );
int mvwadd_wch( WINDOW *win, int y, int x, const cchar_t *wch );

int echo_wchar( const cchar_t *wch );
int wecho_wchar( WINDOW *win, const cchar_t *wch );
```

`add_wch`、`wadd_wch`、`mvadd_wch`和`mvwadd_wch`函数将复杂字符wch放在给定窗口的当前位置。

这些函数执行换行和特殊字符处理，如下所示:
- 如果指向一个空格字符，则该位置之前的任何字符都将被删除。由wch指定的新字符被放置在该位置，并由wch指定再现。然后光标移动到屏幕上的下一个空格字符。
- 如果引用非空格字符，则该位置之前的所有字符都将保留。其中的非空格字符被添加到空格复合字符中，并且忽略`wch`指定的再现。
- 如果其中的字符部分是制表符、换行符、退格符或其他控制字符，则窗口将被更新，光标将像调用`addch`一样移动。

`echo_wchar` 等价于 `add_wch` **+** `refresh`

## add_wchstr

在窗口中添加**复杂字符串**

```c
#include <curses.h>

int add_wchstr(const cchar_t *wchstr);
int add_wchnstr(const cchar_t *wchstr, int n);
int wadd_wchstr(WINDOW * win, const cchar_t *wchstr);
int wadd_wchnstr(WINDOW * win, const cchar_t *wchstr, int n);

int mvadd_wchstr(int y, int x, const cchar_t *wchstr);
int mvadd_wchnstr(int y, int x, const cchar_t *wchstr, int n);
int mvwadd_wchstr(WINDOW *win, int y, int x, const cchar_t *wchstr);
int mvwadd_wchnstr(WINDOW *win, int y, int x, const cchar_t *wchstr, int n);
```

这些函数从当前光标位置开始将(以null结尾的)复杂字符数组wchstr复制到窗口图像结构中。以n为最后一个参数的四个函数最多复制n个元素，但不会超过一行的容量。如果n=-1，则复制整个数组，直到该行能容纳的最大字符数。

## addch

将字符显示到指定位置

```c
#include <curses.h>

int addch(const chtype ch);
int waddch(WINDOW *win, const chtype ch);
int mvaddch(int y, int x, const chtype ch);
int mvwaddch(WINDOW *win, int y, int x, const chtype ch);

int echochar(const chtype ch);
int wechochar(WINDOW *win, const chtype ch);
```

## addchstr

将字符串显示到指定位置

```c
#include <curses.h>

int addchstr(const chtype *chstr);
int addchnstr(const chtype *chstr, int n);
int waddchstr(WINDOW *win, const chtype *chstr);
int waddchnstr(WINDOW *win, const chtype *chstr, int n);

int mvaddchstr(int y, int x, const chtype *chstr);
int mvaddchnstr(int y, int x, const chtype *chstr, int n);
int mvwaddchstr(WINDOW *win, int y, int x, const chtype *chstr);
int mvwaddchnstr(WINDOW *win, int y, int x, const chtype *chstr, int n);
```

## addstr

```c
#include <curses.h>

int addstr(const char *str);
int addnstr(const char *str, int n);
int waddstr(WINDOW *win, const char *str);
int waddnstr(WINDOW *win, const char *str, int n);

int mvaddstr(int y, int x, const char *str);
int mvaddnstr(int y, int x, const char *str, int n);
int mvwaddstr(WINDOW *win, int y, int x, const char *str);
int mvwaddnstr(WINDOW *win, int y, int x, const char *str, int n);
```

## addwstr

```c
#include <curses.h>

int addwstr(const wchar_t *wstr);
int addnwstr(const wchar_t *wstr, int n);
int waddwstr(WINDOW *win, const wchar_t *wstr);
int waddnwstr(WINDOW *win, const wchar_t *wstr, int n);

int mvaddwstr(int y, int x, const wchar_t *wstr);
int mvaddnwstr(int y, int x, const wchar_t *wstr, int n);
int mvwaddwstr(WINDOW *win, int y, int x, const wchar_t *wstr);
int mvwaddnwstr(WINDOW *win, int y, int x, const wchar_t *wstr, int n);
```

## attr

字体和窗口属性

```c
#include <curses.h>

int attr_get(attr_t *attrs, short *pair, void *opts);
int wattr_get(WINDOW *win, attr_t *attrs, short *pair, void *opts);
int attr_set(attr_t attrs, short pair, void *opts);
int wattr_set(WINDOW *win, attr_t attrs, short pair, void *opts);

int attr_off(attr_t attrs, void *opts);
int wattr_off(WINDOW *win, attr_t attrs, void *opts);
int attr_on(attr_t attrs, void *opts);
int wattr_on(WINDOW *win, attr_t attrs, void *opts);

int attroff(int attrs);
int wattroff(WINDOW *win, int attrs);
int attron(int attrs);
int wattron(WINDOW *win, int attrs);
int attrset(int attrs);
int wattrset(WINDOW *win, int attrs);

int chgat(int n, attr_t attr, short pair, const void *opts);
int wchgat(WINDOW *win, int n, attr_t attr, short pair, const void *opts);
int mvchgat(int y, int x, int n, attr_t attr, short pair, const void *opts);
int mvwchgat(WINDOW *win, int y, int x, int n, attr_t attr, short pair, const void *opts);

int color_set(short pair, void* opts);
int wcolor_set(WINDOW *win, short pair, void* opts);

int standend(void);
int wstandend(WINDOW *win);
int standout(void);
int wstandout(WINDOW *win);
```

## beep

响铃

```c
#include <curses.h>

int beep(void);
int flash(void);
```
beep和flash例程用于提醒终端用户。如果可能的话，例行的哔哔声在终端上发出声音警报;否则它会闪烁屏幕(可见的铃铛)。常规闪光闪烁屏幕，如果这是不可能的，声音警报。如果两个警报都不可能，什么都不会发生。几乎所有的终端机都有声音警报(铃声或嘟嘟声)，但只有一些终端机可以在屏幕上闪烁。

## bkgd

窗口背景操作

```c
#include <curses.h>

void bkgdset(chtype ch);
void wbkgdset(WINDOW *win, chtype ch);

int bkgd(chtype ch);
int wbkgd(WINDOW *win, chtype ch);

chtype getbkgd(WINDOW *win);
```

### bkgdset

bkgdset和wbkgdset例程设置窗口的背景。窗口的背景是一个chtype，由任意属性(即再现)和字符的组合组成:
- 背景的属性部分与使用waddch写入窗口的所有非空白字符合并
- 背景的字符和属性部分都与写入窗口的空白字符相结合

背景成为每个字符的属性，并随着字符的滚动和插入/删除行/字符操作而移动。

在特定终端上，背景的属性部分尽可能显示为屏幕上字符的图形再现。

### bkgd

bkgd和wbkgd函数设置当前或指定窗口的背景属性，然后将此设置应用于该窗口中的每个字符位置。根据《X/Open Curses》，它应该这样做:
- 屏幕上的每个字符的呈现都变成了新的背景呈现。
- 只要前一个背景字符出现，它就被更改为新的背景字符。

## bkgrnd

窗口复杂的后台操作例程

```c

#include <curses.h>

int bkgrnd( const cchar_t *wch);
int wbkgrnd( WINDOW *win, const cchar_t *wch);

void bkgrndset(const cchar_t *wch );
void wbkgrndset(WINDOW *win, const cchar_t *wch);

int getbkgrnd(cchar_t *wch);
int wgetbkgrnd(WINDOW *win, cchar_t *wch);
```

## border_set

使用复杂字符和再现的边框或线条

```c
#include <curses.h>

int border_set(const cchar_t *ls, const cchar_t *rs, const cchar_t *ts, const cchar_t *bs, const cchar_t *tl, const cchar_t *tr, const cchar_t *bl, const cchar_t *br );
int wborder_set(WINDOW *win, const cchar_t *ls, const cchar_t *rs, const cchar_t *ts, const cchar_t *bs, const cchar_t *tl, const cchar_t *tr, const cchar_t *bl, const cchar_t *br);
int box_set(WINDOW *win, const cchar_t *verch, const cchar_t *horch);
int hline_set(const cchar_t *wch, int n);
int whline_set(WINDOW *win, const cchar_t *wch, int n);
int mvhline_set(int y, int x, const cchar_t *wch, int n);
int mvwhline_set(WINDOW *win, int y, int x, const cchar_t *wch, int n);
int vline_set(const cchar_t *wch, int n);
int wvline_set(WINDOW *win, const cchar_t *wch, int n);
int mvvline_set(int y, int x, const cchar_t *wch, int n);
int mvwvline_set(WINDOW *win, int y, int x, const cchar_t *wch, int n);
```

border_set和wborder_set函数围绕当前或指定窗口的边缘绘制边框。这些函数不会改变光标位置，也不会自动换行。

除了窗口，每个参数都是一个带有attributes的复杂字符:

    ls - left side,
    rs - right side,
    ts - top side,
    bs - bottom side,
    tl - top left-hand corner,
    tr - top right-hand corner,
    bl - bottom left-hand corner, and
    br - bottom right-hand corner.

如果这些参数中的任何一个为零，则使用相应的默认值(在curses.h中定义):

    WACS_VLINE,
    WACS_VLINE,
    WACS_HLINE,
    WACS_HLINE,
    WACS_ULCORNER,
    WACS_URCORNER,
    WACS_LLCORNER, and
    WACS_LRCORNER.

## border

创建 border —— 水平线或垂直线

```c
#include <curses.h>

int border(chtype ls, chtype rs, chtype ts, chtype bs, chtype tl, chtype tr, chtype bl, chtype br);
int wborder(WINDOW *win, chtype ls, chtype rs, chtype ts, chtype bs, chtype tl, chtype tr, chtype bl, chtype br);

int box(WINDOW *win, chtype verch, chtype horch);

int hline(chtype ch, int n);
int whline(WINDOW *win, chtype ch, int n);
int vline(chtype ch, int n);
int wvline(WINDOW *win, chtype ch, int n);

int mvhline(int y, int x, chtype ch, int n);
int mvwhline(WINDOW *win, int y, int x, chtype ch, int n);
int mvvline(int y, int x, chtype ch, int n);
int mvwvline(WINDOW *win, int y, int x, chtype ch, int n);
```

border, wborder和box例程围绕窗口的边缘绘制一个方框。除了窗口，每个参数都是一个带有at的字符:
    ls - left side,
    rs - right side,
    ts - top side,
    bs - bottom side,
    tl - top left-hand corner,
    tr - top right-hand corner,
    bl - bottom left-hand corner, and
    br - bottom right-hand corner.

如果这些参数中的任何一个为零，则使用相应的默认值(在curses.h中定义):
    ACS_VLINE,
    ACS_VLINE,
    ACS_HLINE,
    ACS_HLINE,
    ACS_ULCORNER,
    ACS_URCORNER,
    ACS_LLCORNER,
    ACS_LRCORNER.

## clear
```c
#include <curses.h>

int erase(void);
int werase(WINDOW *win);

int clear(void);
int wclear(WINDOW *win);

int clrtobot(void);
int wclrtobot(WINDOW *win);

int clrtoeol(void);
int wclrtoeol(WINDOW *win);
```

### erase/werase

erase和werase例程将空格复制到window的每个位置，清除屏幕。
由擦除创建的空白将当前的背景再现(由wbkgdset(3x)设置)合并到其中。

### clear/wclear

clear和wclear例程类似于erase和werase，但它们也调用clearok(3x)，以便在下次调用该窗口的wrefresh时完全清除屏幕并从头重新绘制。

### clrtobot/wclrtobot

clrtobot和wclrtobot例程从光标擦除到屏幕末端。也就是说，它们会擦除窗口中光标下方的所有行。此外，光标右侧的当前行(包括在内)将被擦除。

### clrtoeol/wclrtoeol

clrtool和wclrtool例程擦除光标右侧的当前行(包括当前行末尾)。

## color

```c
#include <curses.h>

int start_color(void);

bool has_colors(void);
bool can_change_color(void);

int init_pair(short pair, short f, short b);
int init_color(short color, short r, short g, short b);
/* extensions */
int init_extended_pair(int pair, int f, int b);
int init_extended_color(int color, int r, int g, int b);

int color_content(short color, short *r, short *g, short *b);
int pair_content(short pair, short *f, short *b);
/* extensions */
int extended_color_content(int color, int *r, int *g, int *b);
int extended_pair_content(int pair, int *f, int *b);

/* extensions */
void reset_color_pairs(void);

int COLOR_PAIR(int n);
PAIR_NUMBER(attrs);
```
Curses支持具有该功能的终端上的颜色属性。要使用这些例程，必须调用start_color，通常在initscr之后。颜色总是成对使用(称为颜色对)。

颜色对由前景色(用于字符)和背景色(用于显示字符的空白字段)组成。

程序员使用init_pair例程初始化颜色对。在初始化后，COLOR_PAIR(n)可用于将pair转换为显示属性。

如果终端能够重新定义颜色，程序员可以使用init_color例程来更改颜色的定义。

例程has_colors和can_change_color返回TRUE或FALSE，这取决于终端是否具有颜色功能以及程序员是否可以更改颜色。

例程color_content允许程序员以初始化的颜色提取红色、绿色和蓝色组件的数量。

例程pair_content允许程序员找出给定颜色对当前是如何定义的。

## delch

在窗口中删除光标下的字符

```c
#include <curses.h>

int delch(void);
int wdelch(WINDOW *win);
int mvdelch(int y, int x);
int mvwdelch(WINDOW *win, int y, int x);
```

这些例程删除光标下的字符;同一行中光标右边的所有字符都向左移动一个位置，最后一个字符为空。光标位置不会改变(移动到y, x(如果指定)之后)。(这并不意味着使用硬件删除字符特性。)

## deleteln

删除并插入多行到windows

```c
#include <curses.h>

int deleteln(void);
int wdeleteln(WINDOW *win);

int insdelln(int n);
int winsdelln(WINDOW *win, int n);

int insertln(void);
int winsertln(WINDOW *win);
```

deleteln和wdeleteln例程删除窗口中光标下的行;当前行以下的所有行都向上移动一行。窗口的底部边框被清除。光标位置不变。

insdelln和winsdelln例程，对于正n，将n行插入到当前行上方的指定窗口中。底线不见了。对于负n，删除n行(从光标下的行开始)，并将剩余的行向上移动。清除底部的n行。当前光标位置保持不变。

insertln和winsertln例程在当前行的上方插入一个空行，而底部的行将丢失。

## extend

```c
#include <curses.h>

const char* curses_version(void);
int use_extended_names(bool enable);
```
这些函数是curses库的扩展，不容易归入其他类别。

## get_wch

从终端键盘获取宽字符

```c
#include <curses.h>

int get_wch(wint_t *wch);
int wget_wch(WINDOW *win, wint_t *wch);
int mvget_wch(int y, int x, wint_t *wch);
int mvwget_wch(WINDOW *win, int y, int x, wint_t *wch);

int unget_wch(const wchar_t wch);
```

## get_wstr

从curses终端键盘获取宽字符数组

```c
#include <curses.h>

int get_wstr(wint_t *wstr);
int getn_wstr(wint_t *wstr, int n);
int wget_wstr(WINDOW *win, wint_t *wstr);
int wgetn_wstr(WINDOW *win, wint_t *wstr, int n);

int mvget_wstr(int y, int x, wint_t *wstr);
int mvgetn_wstr(int y, int x, wint_t *wstr, int n);
int mvwget_wstr(WINDOW *win, int y, int x, wint_t *wstr);
int mvwgetn_wstr(WINDOW *win, int y, int x, wint_t *wstr, int n);
```
get_wstr的效果就像对get_wch(3x)进行了一系列调用，直到处理换行符、其他行结束符或文件结束符条件。文件结束条件由WEOF表示，如<wchar.h>中所定义。换行符和行尾条件由\n wchar_t值表示。在所有实例中，字符串的结尾都以空wchar_t结束。该例程将结果值放在wstr所指向的区域中。

用户的擦除和终止字符将被解释。如果窗口的键盘模式是打开的，KEY_LEFT和KEY_BACKSPACE都被认为等同于用户的终止字符。

只有当前打开echo时，输入的字符才会被返回。在这种情况下，退格回显为删除前一个字符(通常是向左移动)。

wget_wstr的效果就像对wget_wch进行了一系列调用。

mvget_wstr的效果就像调用了一个move，然后调用了一系列get_wch。

mvwget_wstr的效果就像调用wmove，然后调用一系列wget_wch。

getn_wstr, mvgettn_wstr, mvwget_wstr和wget_wstr函数分别与get_wstr, mvget_wstr, mvwget_wstr函数相同，除了*n_*版本最多读取n个字符，让应用程序防止输入缓冲区溢出。

## getcchar

从cchar_t对象获取宽字符串并进行再现，或者从宽字符串对象设置cchar_t对象

```c
#include <curses.h>

int getcchar(const cchar_t *wcval, wchar_t *wch, attr_t *attrs, short *color_pair, void *opts );
int setcchar(cchar_t *wcval, const wchar_t *wch, const attr_t attrs, short color_pair, const void *opts );
```

## getch

从终端键盘获取字符

```c
#include <curses.h>

int getch(void);
int wgetch(WINDOW *win);

int mvgetch(int y, int x);
int mvwgetch(WINDOW *win, int y, int x);

int ungetch(int ch);

/* extension */
int has_key(int ch);
```

## getstr

```c
#include <curses.h>

int getstr(char *str);
int getnstr(char *str, int n);
int wgetstr(WINDOW *win, char *str);
int wgetnstr(WINDOW *win, char *str, int n);

int mvgetstr(int y, int x, char *str);
int mvwgetstr(WINDOW *win, int y, int x, char *str);
int mvgetnstr(int y, int x, char *str, int n);
int mvwgetnstr(WINDOW *win, int y, int x, char *str, int n);
```

## in_wch

这些函数从命名窗口的当前位置提取复杂字符到wcval引用的cchar_t对象中。

```c
#include <curses.h>

int in_wch(cchar_t *wcval);
int win_wch(WINDOW *win, cchar_t *wcval);

int mvin_wch(int y, int x, cchar_t *wcval);
int mvwin_wch(WINDOW *win, int y, int x, cchar_t *wcval);
```

## in_wchstr

```c
#include <curses.h>

int in_wchstr(cchar_t *wchstr);
int in_wchnstr(cchar_t *wchstr, int n);
int win_wchstr(WINDOW *win, cchar_t *wchstr);
int win_wchnstr(WINDOW *win, cchar_t *wchstr, int n);

int mvin_wchstr(int y, int x, cchar_t *wchstr);
int mvin_wchnstr(int y, int x, cchar_t *wchstr, int n);
int mvwin_wchstr(WINDOW *win, int y, int x, cchar_t *wchstr);
int mvwin_wchnstr(WINDOW *win, int y, int x, cchar_t *wchstr, int n);
```

这些函数在wchstr中返回一个复杂字符数组，从指定窗口中的当前光标位置开始。属性(再现)与字符一起存储。

## inch

从窗口获取一个字符和属性

```c
#include <curses.h>

chtype inch(void);
chtype winch(WINDOW *win);

chtype mvinch(int y, int x);
chtype mvwinch(WINDOW *win, int y, int x);
```

## inchstr

从curses窗口获取字符串(和属性)

```c
#include <curses.h>

int inchstr(chtype *chstr);
int inchnstr(chtype *chstr, int n);
int winchstr(WINDOW *win, chtype *chstr);
int winchnstr(WINDOW *win, chtype *chstr, int n);

int mvinchstr(int y, int x, chtype *chstr);
int mvinchnstr(int y, int x, chtype *chstr, int n);
int mvwinchstr(WINDOW *win, int y, int x, chtype *chstr);
int mvwinchnstr(WINDOW *win, int y, int x, chtype *chstr, int n);
```
这些例程返回一个以null结束的chtype数量数组，从指定窗口中的当前光标位置开始，到窗口的右边缘结束。以n作为最后一个参数的四个函数，返回一个最多n个字符的前导子字符串(不包括结尾(chtype)0)。<curhes.h>中定义的常量可以与&(逻辑与)操作符一起使用，从chstr中的任何位置单独提取字符或属性[参见curs_inch(3x)]。

## initscr

屏幕初始化和操作函数

```c
#include <curses.h>

WINDOW *initscr(void);
int endwin(void);

bool isendwin(void);

SCREEN *newterm(const char *type, FILE *outfd, FILE *infd);
SCREEN *set_term(SCREEN *new);
void delscreen(SCREEN* sp);
```
initscr通常是初始化程序时调用的第一个curses例程。有时需要在它之前调用一些特殊的例程;它们是slk_init(3x)， filter, ripoffline, use_env。对于多终端应用，可能会在initscr之前调用newterm。

initscr代码确定终端类型并初始化所有curses数据结构。Initscr还会导致第一次调用刷新(3x)以清除屏幕。如果发生错误，initscr将适当的错误消息写入标准错误并退出;否则，返回指向stdscr的指针。

## inopts

输入选项

```c
#include <curses.h>

int cbreak(void);
int nocbreak(void);

int echo(void);
int noecho(void);

int intrflush(WINDOW *win, bool bf);
int keypad(WINDOW *win, bool bf);
int meta(WINDOW *win, bool bf);
int nodelay(WINDOW *win, bool bf);
int notimeout(WINDOW *win, bool bf);

int nl(void);
int nonl(void);

int raw(void);
int noraw(void);

void qiflush(void);
void noqiflush(void);

int halfdelay(int tenths);
void timeout(int delay);
void wtimeout(WINDOW *win, int delay);

int typeahead(int fd);
```

ncurses库提供了几个函数，可以让应用程序改变终端输入的处理方式。有些是全局的，适用于所有窗口。其他的只应用于特定的窗口。特定于窗口的设置不会自动应用于新窗口或派生窗口。如果需要相同的行为，应用程序必须将这些应用到每个窗口。

### cbreak/nocbreak

通常，tty驱动程序会缓冲键入的字符，直到键入换行符或回车符。cbreak例程禁用行缓冲和删除/终止字符处理(中断和流控制字符不受影响)，使用户键入的字符立即对程序可用。nocbreak例程将终端返回到正常模式。

最初终端可能处于cbreak模式，也可能不处于cbreak模式，因为该模式是继承的;因此，程序应该显式地调用cbreak或nocbreak。大多数使用curses的交互程序都设置了cbreak模式。

> 注意，cbreak会覆盖raw。[参见curs_getch(3x)了解这些例程如何与echo和noecho交互。]

### echo/noecho

echo和noecho例程控制用户输入的字符是否在输入时由getch(3x)回显。tty驱动程序的回显总是被禁用的，但是最初getch处于回显模式，因此输入的字符被回显。大多数交互程序的作者更喜欢在屏幕的一个受控区域中进行自己的回显，或者根本不回显，所以他们通过调用noecho来禁用回显。[参见curs_getch(3x)了解这些例程如何与cbreak和nocbreak交互。]

### halfdelay

half-delay例程用于半延迟模式，它类似于cbreak模式，用户输入的字符可以立即被程序使用。但是，在阻塞了十分之一秒之后，如果没有输入任何东西，则返回ERR。十分位数必须为1到255之间的数字。使用nocbreak离开半延迟模式。

### intrflush

如果启用了intrflush选项(bf为TRUE)，并且按下了键盘上的中断键(interrupt, break, quit)， tty驱动队列中的所有输出都将被刷新，从而对中断做出更快的响应，但会导致curses对屏幕上的内容有错误。禁用该选项(bf为FALSE)可防止刷新。该选项的默认值继承自tty驱动程序设置。窗口参数被忽略。

### keypad

小键盘选项启用用户终端的小键盘。如果启用(bf为TRUE)，用户可以按下一个功能键(如方向键)，wgetch(3x)返回一个代表功能键的值，如KEY_LEFT。如果禁用(bf为FALSE)， curses不会特别处理功能键，程序必须自己解释转义序列。如果终端中的小键盘可以打开(用于传输)和关闭(用于本地工作)，则打开此选项将导致在调用wgetch(3x)时打开终端小键盘。“键盘”的默认值为“FALSE”。

### meta

最初，终端在输入时返回7位还是8位有效位取决于tty驱动程序的控制模式[参见termios(3)]。强制返回8位，调用meta(win, TRUE);在POSIX下，这相当于在终端上设置CS8标志。强制返回7位，调用meta(win, FALSE);在POSIX下，这相当于在终端上设置CS7标志。窗口参数win总是被忽略。如果为终端定义了terminfo能力smm (meta_on)和rmm (meta_off)，则在调用meta(win, TRUE)时向终端发送smm，在调用meta(win, FALSE)时向终端发送rmm。

### nl/nonl

nl和nonl例程控制底层显示设备是否在输入时将返回键转换为换行符。

### nodelay

nodelay选项使getch成为一个非阻塞调用。如果没有准备好的输入，getch返回ERR。如果被禁用(bf为FALSE)， getch将等待按键被按下。

### notimeout

当解释转义序列时，wgetch(3x)在等待下一个字符时设置一个计时器。如果调用notimeout(win, TRUE)，则wgetch不设置定时器。该超时的目的是区分从功能键接收到的序列和用户键入的序列。


### raw/noraw

raw和noraw例程将终端设置为raw模式或退出raw模式。Raw模式类似于cbreak模式，输入的字符会立即传递给用户程序。不同之处在于，在原始模式中，中断、退出、挂起和流控制字符都是未经解释传递的，而不是生成信号。BREAK键的行为取决于tty驱动程序中不是由curses设置的其他位。

### qiflush/noqiflush

当使用noqiflush例程时，与INTR、QUIT和SUSP字符相关的输入和输出队列的正常刷新将不会进行[参见termios(3)]。当调用qiflush时，当读取这些控制字符时，队列将被刷新。如果希望在信号处理程序退出后继续输出，就像中断没有发生一样，那么可能需要在信号处理程序中调用noqiflush。

### timeout/wtimeout

timeout和wtimeout例程为给定窗口设置阻塞或非阻塞读取。如果delay为负，则使用阻塞读取(即无限期地等待输入)。如果延迟为零，则使用非阻塞读取(即，如果没有输入等待，则read返回ERR)。如果delay为正，则读取延迟毫秒的块，如果仍然没有输入则返回ERR。因此，这些例程提供了与nodelay相同的功能，加上能够仅阻塞延迟毫秒(其中delay为正)的额外功能。

### typeahead

curses库通过在更新屏幕时定期查找提前输入来进行“换行优化”。如果找到了来自tty的输入，则当前更新将被推迟，直到再次调用refresh(3x)或doupdate。这样可以更快地响应预先键入的命令。通常，传递给newterm的输入FILE指针(在使用initscr的情况下是stdin)将被用于执行输入前检查。typeahead例程指定使用文件描述符fd来检查typeahead。如果fd为-1，则不执行输入前检查。

## ins_wch

在屏幕上插入复杂字符串

```c
#include <curses.h>

int ins_wch(const cchar_t *wch);
int wins_wch(WINDOW *win, const cchar_t *wch);

int mvins_wch(int y, int x, const cchar_t *wch);
int mvwins_wch(WINDOW *win, int y, int x, const cchar_t *wch);
```
这些例程，在光标下的字符之前插入带有再现的复杂字符。光标右边的所有字符都向右移动一个空格，有可能丢失行中最右边的字符。插入操作不会改变游标位置。

## ins_wstr

在屏幕上插入宽字符

```c
#include <curses.h>

int ins_wstr(const wchar_t *wstr);
int ins_nwstr(const wchar_t *wstr, int n);
int wins_wstr(WINDOW *win, const wchar_t *wstr);
int wins_nwstr(WINDOW *win, const wchar_t *wstr, int n);

int mvins_wstr(int y, int x, const wchar_t *wstr);
int mvins_nwstr(int y, int x, const wchar_t *wstr, int n);
int mvwins_wstr(WINDOW *win, int y, int x, const wchar_t *wstr);
int mvwins_nwstr(WINDOW *win, int y, int x, const wchar_t *wstr, int n);
```
这些例程在光标下的字符之前插入一个wchar_t字符串(一行中可以容纳多少字符)。光标右边的所有字符都向右移动，有可能丢失行中最右边的字符。不执行包装。光标位置不会改变(移动到y, x(如果指定)之后)。以n作为最后一个参数的四个例程插入一个最多n个wchar_t字符的前导子字符串。如果n小于1，则插入整个字符串。

如果wstr中的字符是制表符、换行符、回车符或退格符，则光标将在窗口内适当移动。换行符在移动之前也会执行一个clrtool。制表符被认为是每八列。如果wstr中的字符是另一个控制字符，则以^X表示法绘制。在添加一个控制字符之后调用win_wch(如果需要的话，移动到它)不会返回控制字符，而是返回控制字符的^-表示中的一个字符。

## insch

游标前插入一个字符串

```c
#include <curses.h>

int insch(chtype ch);
int winsch(WINDOW *win, chtype ch);

int mvinsch(int y, int x, chtype ch);
int mvwinsch(WINDOW *win, int y, int x, chtype ch);
```
这些例程在光标下的字符之前插入字符ch。光标右边的所有字符都向右移动一个空格，有可能丢失行中最右边的字符。插入操作不会改变游标位置。

## insstr

游标前插入字符串

```c
#include <curses.h>

int insstr(const char *str);
int insnstr(const char *str, int n);
int winsstr(WINDOW *win, const char *str);
int winsnstr(WINDOW *win, const char *str, int n);

int mvinsstr(int y, int x, const char *str);
int mvinsnstr(int y, int x, const char *str, int n);
int mvwinsstr(WINDOW *win, int y, int x, const char *str);
int mvwinsnstr(WINDOW *win, int y, int x, const char *str, int n);
```
这些例程在光标下的字符之前插入一个字符串(一行可以容纳多少字符)。光标右边的所有字符都向右移动，可能会丢失行中最右边的字符。光标位置不会改变(移动到y, x(如果指定)之后)。以n作为最后一个参数的函数插入最多n个字符的前导子字符串。如果n<=0，则插入整个字符串。

特殊字符在addch中被处理。

## instr

获取游标处的字符

```c
#include <curses.h>

int instr(char *str);
int innstr(char *str, int n);
int winstr(WINDOW *win, char *str);
int winnstr(WINDOW *win, char *str, int n);

int mvinstr(int y, int x, char *str);
int mvinnstr(int y, int x, char *str, int n);
int mvwinstr(WINDOW *win, int y, int x, char *str);
int mvwinnstr(WINDOW *win, int y, int x, char *str, int n);
```
这些例程返回str中的字符串，从指定窗口中的当前光标位置开始提取。从字符中剥离属性。以n作为最后一个参数的四个函数返回一个最多n个字符的前导子字符串(不包括尾随的NUL)。

## inwstr

获取游标处字符串

```c
#include <curses.h>

int inwstr(wchar_t *wstr);
int innwstr(wchar_t *wstr, int n);
int winwstr(WINDOW *win, wchar_t *wstr);
int winnwstr(WINDOW *win, wchar_t *wstr, int n);

int mvinwstr(int y, int x, wchar_t *wstr);
int mvinnwstr(int y, int x, wchar_t *wstr, int n);
int mvwinwstr(WINDOW *win, int y, int x, wchar_t *wstr);
int mvwinnwstr(WINDOW *win, int y, int x, wchar_t *wstr, int n);
```
这些例程在wstr中返回一个wchar_t宽字符的字符串，从指定窗口中的当前光标位置开始提取。

以n为最后一个参数的四个函数返回前导子字符串最多n个字符长(不包括尾随的NUL)。当前行的末尾，或者当有n个字符存储在wstr引用的位置时停止获取。

如果n的大小不足以存储一个完整的复杂字符，则会生成一个错误。

## kernel

```c
#include <curses.h>

int def_prog_mode(void);
int def_shell_mode(void);

int reset_prog_mode(void);
int reset_shell_mode(void);

int resetty(void);
int savetty(void);

void getsyx(int y, int x);
void setsyx(int y, int x);

int ripoffline(int line, int (*init)(WINDOW *, int));
int curs_set(int visibility);
int napms(int ms);
```

下面的例程提供了对各种curses功能的低级访问。这些例程通常在库例程中使用。

### def_prog_mode, def_shell_mode

def_prog_mode和def_shell_mode例程将当前终端模式保存为“程序”(curses)或“shell”(非curses)状态，供reset_prog_mode和reset_shell_mode例程使用。这是由initscr自动完成的。每个由newterm分配的屏幕上下文都有一个这样的保存区域。

### reset_prog_mode, reset_shell_mode

reset_prog_mode和reset_shell_mode例程将终端恢复为“程序”(处于诅咒状态)或“shell”(处于诅咒状态)状态。这些是由endwin(3x)自动完成的，在endwin之后，由doupdate自动完成，所以通常不调用它们。

### resetty, savetty

复位和安全例程保存和恢复终端模式的状态。Savetty将当前状态保存在缓冲区中，重置将状态恢复到最后一次调用Savetty时的状态。

### getsyx

getsyx例程返回虚拟屏幕光标在y和x中的当前坐标。如果leaveok当前为TRUE，则返回-1，-1。如果线条已经从屏幕顶部移除，使用ripoffline, y和x包括这些线条;因此，y和x只能作为setsyx的参数。

很少有应用程序会使用这个特性，大多数应用程序会使用getyx。

### setsyx

setsyx例程将虚拟屏幕光标设置为y, x。如果y和x都是-1，则设置leaveok。getsyx和setsyx这两个例程被设计为库例程使用，库例程操作curses窗口，但不想改变程序光标的当前位置。库例程将在开始时调用getsyx，对其自己的窗口进行操作，对其窗口执行wnoutrefresh，调用setsyx，然后调用doupdate。

很少有应用程序会使用这个特性，大多数应用程序会使用wmove。

### ripoffline

ripoffline例程提供了对slk_init[参见curs_slk(3x)]用于减小屏幕大小的相同工具的访问。Ripoffline必须在initscr或newterm调用之前调用，以准备这些初始操作:
- 如果line为正数，则从stdscr顶部删除一行
- 如果line为负数，则从底部删除一行

当最终的初始化在initscr内部完成时，例程init(由用户提供)将被调用，并带有两个参数:
- 一个窗口指针，指向已分配和的单行窗口
- 窗口中列数的整数

在这个初始化例程中，不能保证整型变量LINES和COLS(在<curses.h>中定义)是准确的，也不能调用wrefresh或doupdate。允许在初始化例程期间调用wnoutrefresh。

在调用initscr或newterm之前，ripoffline最多可以被调用五次。

### curs_set

curs_set例程将游标状态分别设置为不可见、正常或非常可见(可见性分别为0、1或2)。如果终端支持所请求的可见性，则返回先前的游标状态;否则返回ERR。

### napms

napms例程用于毫秒级的睡眠

## legacy

获取curses光标和窗口坐标，属性

```c
#include <curses.h>

int getattrs(const WINDOW *win);

int getbegx(const WINDOW *win);
int getbegy(const WINDOW *win);

int getcurx(const WINDOW *win);
int getcury(const WINDOW *win);

int getmaxx(const WINDOW *win);
int getmaxy(const WINDOW *win);

int getparx(const WINDOW *win);
int getpary(const WINDOW *win);
```

这些遗留函数比X/Open Curses函数使用起来更简单:

- getattrs函数返回与wattr_get相同的属性数据。然而，getattrs返回一个整数(实际上是一个chtype)，而wattr_get在一个单独的参数中返回当前颜色对。在宽字符库配置中，颜色对可能不适合chtype，因此wattr_get是获取颜色信息的唯一方法。因为getattrs在单个参数中返回属性，所以应用程序不可能将其与ERR (a -1)区分开来。如果窗口参数为空，getattrs将返回A_NORMAL(零)。
- getbegy和getbegx函数返回与getbegyx相同的数据。
- getcury和getcurx函数返回与getyx相同的数据。
- getmaxx和getmaxx函数返回与getmaxyx相同的数据。
- getpary和getparx函数返回与getparyx相同的数据。

## memleaks

内存检测

```c
#include <curses.h>

void exit_curses(int code);

#include <term.h>

void exit_terminfo(int code);

/* deprecated (intentionally not declared in curses.h or term.h) */
void _nc_freeall(void);
void _nc_free_and_exit(int code);
void _nc_free_tinfo(int code);
```
这些函数用于简化ncurses库中的内存泄漏分析。

任何curses的实现都不能释放与屏幕相关的内存，因为(即使在调用endwin(3x)之后)，它必须在下一次调用refresh(3x)时可用。还有一些内存块是出于性能原因而保留的。这使得很难分析curses应用程序的内存泄漏。当使用ncurses库的特殊配置的调试版本时，应用程序可以调用释放这些内存块的函数，从而简化内存泄漏检查过程。

一些函数以“\_nc\_”前缀命名，因为它们不打算在非调试库中使用:

### _nc_freeall

这将释放(几乎)ncurses分配的所有内存。

### _nc_free_and_exit

这将释放由ncurses分配的内存(如_nc_freeall)，并退出程序。它优先于_nc_freeall，因为可能需要一些内存来保持应用程序运行。简单地退出(使用给定的退出代码)更安全。

### _nc_free_tinfo

如果只使用低级的terminfo函数(以及相应的库)，则使用此函数。像_nc_free_and_exit一样，它在释放内存后退出程序。

前缀为“_nc”的函数通常不可用;它们必须在构建时使用——disable-leaks选项配置到库中。编译代码释放通常不会被释放的内存。

如果库配置为支持内存泄漏检查，exit_curses和exit_terminfo函数将调用_nc_free_and_exit和_nc_free_tinfo。如果库没有配置为支持内存泄漏检查，则会简单地调用exit。

## mouse

```c
#include <curses.h>

typedef unsigned long mmask_t;

typedef struct 
{
    short id;         /* ID to distinguish multiple devices */
    int x, y, z;      /* event coordinates */
    mmask_t bstate;   /* button state bits */
} MEVENT;

bool has_mouse(void);

int getmouse(MEVENT *event);
int ungetmouse(MEVENT *event);

mmask_t mousemask(mmask_t newmask, mmask_t *oldmask);

bool wenclose(const WINDOW *win, int y, int x);

bool mouse_trafo(int* pY, int* pX, bool to_screen);
bool wmouse_trafo(const WINDOW* win, int* pY, int* pX, bool to_screen);

int mouseinterval(int erval);
```

这些函数为来自ncurses(3x)的鼠标事件提供了接口。鼠标事件由wgetch(3x)输入流中的KEY_MOUSE伪键值表示。

## move

移动游标的位置

```c
#include <curses.h>

int move(int y, int x);
int wmove(WINDOW *win, int y, int x);
```
这些例程将与窗口相关联的光标移动到y行和x列。在调用refresh(3x)之前，此例程不会移动终端的物理光标。指定的位置相对于窗口的左上角，即(0,0)。

## opaque

```c
#include <curses.h>

bool is_cleared(const WINDOW *win);
bool is_idcok(const WINDOW *win);
bool is_idlok(const WINDOW *win);
bool is_immedok(const WINDOW *win);
bool is_keypad(const WINDOW *win);
bool is_leaveok(const WINDOW *win);
bool is_nodelay(const WINDOW *win);
bool is_notimeout(const WINDOW *win);
bool is_pad(const WINDOW *win);
bool is_scrollok(const WINDOW *win);
bool is_subwin(const WINDOW *win);
bool is_syncok(const WINDOW *win);
WINDOW * wgetparent(const WINDOW *win);
int wgetdelay(const WINDOW *win);
int wgetscrreg(const WINDOW *win, int *top, int *bottom);
```
该实现提供了返回WINDOW结构中设置的属性的函数，如果定义了符号NCURSES_OPAQUE，则允许其为“opaque”:

    is_cleared: returns the value set in clearok
    is_idcok: returns the value set in idcok
    is_idlok: returns the value set in idlok
    is_immedok: returns the value set in immedok
    is_keypad: returns the value set in keypad
    is_leaveok: returns the value set in leaveok
    is_nodelay: returns the value set in nodelay
    is_notimeout: returns the value set in notimeout
    is_pad: returns TRUE if the window is a pad i.e., created by newpad
    is_scrollok: returns the value set in scrollok
    is_subwin: returns TRUE if the window is a subwindow, i.e., created by subwin or derwin
    is_syncok: returns the value set in syncok
    wgetdelay: returns the delay timeout as set in wtimeout.
    wgetparent: returns the parent WINDOW pointer for subwindows, or NULL for windows having no parent.
    wgetscrreg: returns the top and bottom rows for the scrolling margin as set in wsetscrreg.

## output

输出选项

```c 
#include <curses.h>

int clearok(WINDOW *win, bool bf);
int idlok(WINDOW *win, bool bf);
void idcok(WINDOW *win, bool bf);
void immedok(WINDOW *win, bool bf);
int leaveok(WINDOW *win, bool bf);
int scrollok(WINDOW *win, bool bf);

int setscrreg(int top, int bot);
int wsetscrreg(WINDOW *win, int top, int bot);
```
这些例程设置选项，更改curses中的输出样式。所有选项的初始值都是FALSE，除非另有说明。在调用endwin(3x)之前，没有必要关闭这些选项。

### clearok

如果clearok以TRUE为参数调用，那么下一次调用该窗口的wrefresh将完全清除屏幕并从头重新绘制整个屏幕。这在屏幕内容不确定时很有用，或者在某些情况下为了获得更令人愉悦的视觉效果。如果clearok的win参数是全局变量curscr，那么对任意窗口的下一个wrefresh调用将导致屏幕被清除并从头重新绘制。

### idlok

如果idlok以TRUE作为第二个参数调用，curses会考虑使用这样配置的终端的硬件插入/删除行特性。以FALSE作为第二个参数调用idlok将禁止使用行插入和删除。只有当应用程序需要插入/删除行(例如，用于屏幕编辑器)时，才应该启用此选项。它在默认情况下是禁用的，因为在应用程序中使用插入/删除行时，它在视觉上很讨厌。如果插入/删除行不能使用，curses将重新绘制所有行中已更改的部分。

### idcok

如果idcok以FALSE作为第二个参数调用，curses将不再考虑使用这样配置的终端的硬件插入/删除字符特性。默认情况下启用字符插入/删除功能。以TRUE作为第二个参数调用idcok可以重新使用字符插入和删除。

### immedok

如果以TRUE作为参数调用immedok，那么窗口图像中的任何变化，例如由waddch、wclrtobot、wscrl等引起的变化，都会自动引起对wrefresh的调用。但是，由于反复调用wrefresh，它可能会大大降低性能。默认是禁用的。

### leaveok

通常情况下，硬件游标会停留在刷新窗口游标的位置。leaveok选项允许光标停留在更新恰好停留的地方。它对于不使用游标的应用程序非常有用，因为它减少了游标移动的需要。

### scrollok

scrollok选项控制将窗口的光标移出窗口边缘或滚动区域时发生的情况，无论是由于在底线上执行换行操作，还是由于键入最后一行的最后一个字符。如果禁用，(bf为FALSE)，游标将留在底线上。如果启用(bf为TRUE)，窗口将向上滚动一行(注意，为了在终端上获得物理滚动效果，还需要调用idlok)。

### setscrreg/wsetscrreg

setscrreg和wsetscrreg例程允许应用程序程序员在窗口中设置软件滚动区域。top和bot参数是滚动区域的顶部和底部边缘的行号。(第0行是窗口的顶行。)如果启用了此选项和scrollok，则尝试离开底边距会导致滚动区域中的所有行沿第一行的方向滚动一行。只滚动窗口的文本。(请注意，这与在终端中使用物理滚动区域功能无关，就像在VT100中那样。如果启用了idlok，并且终端具有滚动区域或插入/删除行功能，那么它们可能会被输出例程使用。)

## overlay

```c
#include <curses.h>

int overlay(const WINDOW *srcwin, WINDOW *dstwin);
int overwrite(const WINDOW *srcwin, WINDOW *dstwin);
int copywin(const WINDOW *srcwin, WINDOW *dstwin, int sminrow, int smincol, int dminrow, int dmincol, int dmaxrow, int dmaxcol, int overlay);
```

### overlay, overwrite

overlay和overwrite例程将srcwin覆盖在dstwin之上。scrwin和dstwin的大小不需要相同;只复制两个Windows重叠的文本。区别在于overlay覆盖是非破坏性的(空白不会被复制)，而overwrite覆盖是破坏性的。

### copywin

copywin例程提供了对覆盖例程和覆盖例程更细粒度的控制。与预刷新例程一样，在目标窗口(dminrow, dmincol)和(dmaxrow, dmaxcol)中指定矩形，在源窗口的左上角坐标(sminrow, smincol)中指定矩形。如果参数overlay为真，则复制是非破坏性的，就像overlay一样。

## pad

```c
#include <curses.h>

WINDOW *newpad(int nlines, int ncols);
WINDOW *subpad(WINDOW *orig, int nlines, int ncols, int begin_y, int begin_x);
int prefresh(WINDOW *pad, int pminrow, int pmincol, int sminrow, int smincol, int smaxrow, int smaxcol);
int pnoutrefresh(WINDOW *pad, int pminrow, int pmincol, int sminrow, int smincol, int smaxrow, int smaxcol);
int pechochar(WINDOW *pad, chtype ch);
int pecho_wchar(WINDOW *pad, const cchar_t *wch);
```

### newpad

newpad例程创建并返回一个指针，指向具有给定行数(nlines)和列数(ncols)的新pad数据结构。一个垫子就像一个窗口，除了它不受屏幕大小的限制，并且不一定与屏幕的特定部分相关联。当需要一个大的窗口时，可以使用衬垫，并且一次只会在屏幕上显示窗口的一部分。不会发生pad的自动刷新(例如，从滚动或输入的回显)。

使用pad作为实参调用wrefresh是不合法的;应该调用prefresh或pnoutrefresh例程。注意，这些例程需要额外的参数来指定要显示的pad部分以及用于显示的屏幕上的位置。

### subpad

subpad例程创建并返回一个指针，该指针指向pad内的子窗口，具有给定的行数(nlines)和列数(ncols)。与使用屏幕坐标的subwin不同，窗口位于pad上的位置(begin_x, begin_y)。该窗口位于窗口源的中间，因此对一个窗口所做的更改会影响两个窗口。在使用这个例程的过程中，通常需要在调用prefresh之前调用origo上的touchwin或touchline。

### prefresh, pnouterfresh

prefresh和pnoutrefresh例程类似于wrefresh和wnoutrefresh，只不过它们与pad而不是窗口相关。需要附加的参数来指示涉及到衬垫和屏幕的哪个部分。

- pminrow和pmincol参数指定要显示在pad上的矩形的左上角。
- sminrow、smincol、smaxrow和smaxcol参数指定要在屏幕上显示的矩形的边。

要在pad中显示的矩形的右下角是根据屏幕坐标计算的，因为矩形必须是相同的大小。两个矩形必须完全包含在各自的结构中。pminrow、pmincol、sminrow或smincol的负值被视为零。

### pechochar

pechochar例程在功能上等价于调用addch后调用refresh(3x)，调用waddch后调用wrefresh，或者调用waddch后调用prefresh。考虑到只输出单个字符的知识，对于非控制字符，使用这些例程而不是它们的等量例程可能会看到相当大的性能增益。在pechochar的情况下，屏幕上垫的最后一个位置被重用，用于预刷新参数。

### pecho_wchar

pecho_wchar函数是pechochar的类似宽字符形式。它输出一个字符到一个垫子，并立即刷新垫子。它通过调用wadd_wch，然后调用prefresh来实现这一点。


## print

```c
#include <curses.h>

int mcprint(char *data, int len);
```

该函数使用mc5p或mc4和mc5功能(如果它们存在)将给定的数据传送到连接到终端的打印机。

> 请注意，mcprint代码无法对打印机进行流控制，也无法知道它有多少缓冲。您的应用程序负责将对打印机的写入速率保持在其连续吞吐率以下(通常约为其标称cps等级的一半)。点阵打印机和每分钟6页的激光通常可以处理80cps，所以一个好的保守的经验法则是在输出每80个字符的行之后睡一秒钟。

## printw
```c
#include <curses.h>

int printw(const char *fmt, ...);
int wprintw(WINDOW *win, const char *fmt, ...);
int mvprintw(int y, int x, const char *fmt, ...);
int mvwprintw(WINDOW *win, int y, int x, const char *fmt, ...);
int vw_printw(WINDOW *win, const char *fmt, va_list varglist);

/* obsolete */
int vwprintw(WINDOW *win, const char *fmt, va_list varglist);
```

## refresh
```c
#include <curses.h>

int refresh(void);
int wrefresh(WINDOW *win);
int wnoutrefresh(WINDOW *win);
int doupdate(void);

int redrawwin(WINDOW *win);
int wredrawln(WINDOW *win, int beg_line, int num_lines);
```

## scanw

```c
#include <curses.h>

int scanw(const char *fmt, ...);
int wscanw(WINDOW *win, const char *fmt, ...);
int mvscanw(int y, int x, const char *fmt, ...);
int mvwscanw(WINDOW *win, int y, int x, const char *fmt, ...);

int vw_scanw(WINDOW *win, const char *fmt, va_list varglist);

/* obsolete */
int vwscanw(WINDOW *win, const char *fmt, va_list varglist);
```

## scr_dump

```c
#include <curses.h>

int scr_dump(const char *filename);
int scr_restore(const char *filename);
int scr_init(const char *filename);
int scr_set(const char *filename);
```

## scroll

```c
#include <curses.h>

int scroll(WINDOW *win);

int scrl(int n);
int wscrl(WINDOW *win, int n);
```
scroll例程将窗口向上滚动一行。这涉及到移动窗口数据结构中的行。作为一种优化，如果窗口的滚动区域是整个屏幕，则可以同时滚动物理屏幕。

对于正n, scrl和wscrl例程将窗口向上滚动n行(第i+n行变成第i行);否则将窗口向下滚动n行。这涉及到移动窗口字符图像结构中的行。当前光标位置不变。

为了使这些功能工作，滚动必须通过scrollok启用。

## slk

```c
#include <curses.h>

int slk_init(int fmt);

int slk_set(int labnum, const char *label, int fmt);
int slk_wset(int labnum, const wchar_t *label, int fmt);

char *slk_label(int labnum);

int slk_refresh(void);
int slk_noutrefresh(void);
int slk_clear(void);
int slk_restore(void);
int slk_touch(void);

int slk_attron(const chtype attrs);
int slk_attroff(const chtype attrs);
int slk_attrset(const chtype attrs);
int slk_attr_on(attr_t attrs, void* opts);
int slk_attr_off(const attr_t attrs, void * opts);
int slk_attr_set(const attr_t attrs, short pair, void* opts);
/* extension */
attr_t slk_attr(void);

int slk_color(short pair);
/* extension */
int extended_slk_color(int pair);
```

`slk*`函数操作存在于许多终端上的软功能键标签集。对于那些没有软标签的终端，curses接管了stdscr的底部边框线，减小了stdscr的大小和变量LINES。cruses有八个标签，每个标签最多八个字符。除此之外，ncurses实现还支持一种模式，它模拟12个标签，每个标签最多5个字符。这对于类pc的终端用户设备很有用。Ncurses通过接管屏幕底部的两行来模拟这种模式。

## sp_funcs

```c
#include <curses.h>

int alloc_pair_sp(SCREEN* sp, int fg, int bg);
int assume_default_colors_sp(SCREEN* sp, int fg, int bg);
int baudrate_sp(SCREEN* sp);
int beep_sp(SCREEN* sp);
bool can_change_color_sp(SCREEN* sp);
int cbreak_sp(SCREEN* sp);
int color_content_sp(SCREEN* sp, short color, short* r, short* g, short* b);
int curs_set_sp(SCREEN* sp, int visibility);
int def_prog_mode_sp(SCREEN* sp);
int def_shell_mode_sp(SCREEN* sp);

int define_key_sp(SCREEN* sp, const char * definition, int keycode);
int delay_output_sp(SCREEN* sp, int ms);
int doupdate_sp(SCREEN* sp);
int echo_sp(SCREEN* sp);
int endwin_sp(SCREEN* sp);
char erasechar_sp(SCREEN* sp);
int erasewchar_sp(SCREEN* sp, wchar_t *ch);
int extended_color_content_sp(SCREEN * sp, int color, int * r, int * g, int * b);
int extended_pair_content_sp(SCREEN* sp, int pair, int * fg, int * bg);
int extended_slk_color_sp(SCREEN* sp, int pair);

void filter_sp(SCREEN* sp);
int find_pair_sp(SCREEN* sp, int fg, int bg);
int flash_sp(SCREEN* sp);
int flushinp_sp(SCREEN* sp);
int free_pair_sp(SCREEN* sp, int pair);
int get_escdelay_sp(SCREEN* sp);
int getmouse_sp(SCREEN* sp, MEVENT* event);
WINDOW* getwin_sp(SCREEN* sp, FILE* filep);
int halfdelay_sp(SCREEN* sp, int tenths);
bool has_colors_sp(SCREEN* sp);

bool has_ic_sp(SCREEN* sp);
bool has_il_sp(SCREEN* sp);
int has_key_sp(SCREEN* sp, int ch);
bool has_mouse_sp(SCREEN* sp);
int init_color_sp(SCREEN* sp, short color, short r, short g, short b);
int init_extended_color_sp(SCREEN* sp, int color, int r, int g, int b);
int init_extended_pair_sp(SCREEN* sp, int pair, int fg, int bg);
int init_pair_sp(SCREEN* sp, short pair, short fg, short bg);
int intrflush_sp(SCREEN* sp, WINDOW* win, bool bf);
bool is_term_resized_sp(SCREEN* sp, int lines, int columns);

bool isendwin_sp(SCREEN* sp);
int key_defined_sp(SCREEN* sp, const char *definition);
char* keybound_sp(SCREEN* sp, int keycode, int count);
NCURSES_CONST char * keyname_sp(SCREEN* sp, int c);
int keyok_sp(SCREEN* sp, int keycode, bool enable);
char killchar_sp(SCREEN* sp);
int killwchar_sp(SCREEN* sp, wchar_t *ch);
char* longname_sp(SCREEN* sp);
int mcprint_sp(SCREEN* sp, char *data, int len);
int mouseinterval_sp(SCREEN* sp, int erval);

mmask_t mousemask_sp(SCREEN* sp, mmask_t newmask, mmask_t *oldmask);
int mvcur_sp(SCREEN* sp, int oldrow, int oldcol, int newrow, int newcol);
int napms_sp(SCREEN* sp, int ms);
WINDOW* newpad_sp(SCREEN* sp, int nrows, int ncols);
SCREEN* new_prescr(void);
SCREEN* newterm_sp(SCREEN* sp, const char *type, FILE *outfd, FILE *infd);
WINDOW* newwin_sp(SCREEN* sp, int nlines, int ncols, int begin_y, int begin_x);
int nl_sp(SCREEN* sp);
int nocbreak_sp(SCREEN* sp);
int noecho_sp(SCREEN* sp);

void nofilter_sp(SCREEN* sp);
int nonl_sp(SCREEN* sp);
void noqiflush_sp(SCREEN* sp);
int noraw_sp(SCREEN* sp);
int pair_content_sp(SCREEN* sp, short pair, short* fg, short* bg);
void qiflush_sp(SCREEN* sp);
int raw_sp(SCREEN* sp);
int reset_prog_mode_sp(SCREEN* sp);
void reset_color_pairs_sp(SCREEN* sp);
int reset_shell_mode_sp(SCREEN* sp);

int resetty_sp(SCREEN* sp);
int resize_term_sp(SCREEN* sp, int lines, int columns);
int resizeterm_sp(SCREEN* sp, int lines, int columns);
int ripoffline_sp(SCREEN* sp, int line, int (*init)(WINDOW* win, int fmt));
int savetty_sp(SCREEN* sp);
int scr_init_sp(SCREEN* sp, const char *filename);
int scr_restore_sp(SCREEN* sp, const char *filename);
int scr_set_sp(SCREEN* sp, const char *filename);
int set_escdelay_sp(SCREEN* sp, int ms);
int set_tabsize_sp(SCREEN* sp, int cols);

int slk_attr_set_sp(SCREEN* sp, const attr_t attrs, short pair, void*opts);
int slk_attrset_sp(SCREEN* sp, const chtype a);
int slk_attroff_sp(SCREEN* sp, const chtype a);
int slk_attron_sp(SCREEN* sp, const chtype a);
attr_t slk_attr_sp(SCREEN* sp);
int slk_clear_sp(SCREEN* sp);
int slk_color_sp(SCREEN* sp, short pair);
int slk_init_sp(SCREEN* sp, int fmt);
char* slk_label_sp(SCREEN* sp, int labnum);
int slk_noutrefresh_sp(SCREEN* sp);

int slk_refresh_sp(SCREEN* sp);
int slk_restore_sp(SCREEN* sp);
int slk_set_sp(SCREEN* sp, int labnum, const char * label, int fmt);
int slk_touch_sp(SCREEN* sp);
int start_color_sp(SCREEN* sp);
attr_t term_attrs_sp(SCREEN* sp);
chtype termattrs_sp(SCREEN* sp);
char* termname_sp(SCREEN* sp);
int typeahead_sp(SCREEN* sp, int fd);
int unget_wch_sp(SCREEN* sp, const wchar_t wch);

int ungetch_sp(SCREEN* sp, int ch);
int ungetmouse_sp(SCREEN* sp,MEVENT * event);
int use_default_colors_sp(SCREEN* sp);
void use_env_sp(SCREEN* sp, bool bf);
int use_legacy_coding_sp(SCREEN* sp, int level);
void use_tioctl_sp(SCREEN *sp, bool bf);
int vid_attr_sp(SCREEN* sp, attr_t attrs, short pair, void * opts);
int vid_puts_sp(SCREEN* sp, attr_t attrs, short pair, void * opts, NCURSES_SP_OUTC putc);
int vidattr_sp(SCREEN* sp, chtype attrs);
int vidputs_sp(SCREEN* sp, chtype attrs, NCURSES_SP_OUTC putc);
wchar_t* wunctrl_sp(SCREEN* sp, cchar_t *ch);

#include <form.h>

FORM* new_form_sp(SCREEN* sp, FIELD **fields);

#include <menu.h>

MENU* new_menu_sp(SCREEN* sp, ITEM **items);

#include <panel.h>

PANEL* ceiling_panel(SCREEN* sp);
PANEL* ground_panel(SCREEN* sp);
void update_panels_sp(SCREEN* sp);

#include <term.h>

int del_curterm_sp(SCREEN* sp, TERMINAL *oterm);
int putp_sp(SCREEN* sp, const char *str);
int restartterm_sp(SCREEN* sp, NCURSES_CONST char*term, int filedes, int *errret);
TERMINAL* set_curterm_sp(SCREEN* sp, TERMINAL*nterm);
int tgetent_sp(SCREEN* sp, char *bp, const char *name);
int tgetflag_sp(SCREEN* sp, const char *capname);
int tgetnum_sp(SCREEN* sp, const char *capname);
char* tgetstr_sp(SCREEN* sp, const char *capname, char **area);
char* tgoto_sp(SCREEN* sp, const char *capname, int col, int row);
int tigetflag_sp(SCREEN* sp, const char *capname);
int tigetnum_sp(SCREEN* sp, const char *capname);
char* tigetstr_sp(SCREEN* sp, const char *capname);
/* may instead use 9 long parameters */
char* tparm_sp(SCREEN* sp, const char *str, ...);
int tputs_sp(SCREEN* sp, const char *str, int affcnt, NCURSES_SP_OUTC putc);

#include <unctrl.h>

NCURSES_CONST char* unctrl_sp(SCREEN* sp, chtype c);
```
可以对该实现进行配置，以提供一组功能，从而提高管理多个屏幕的能力。该特性可以添加到ncurses支持的任何配置中;它添加了新的入口点，而不改变任何现有入口点的含义。

## termattrs

```c
#include <curses.h>

int baudrate(void);
char erasechar(void);
int erasewchar(wchar_t *ch);
bool has_ic(void);
bool has_il(void);
char killchar(void);
int killwchar(wchar_t *ch);
char *longname(void);
attr_t term_attrs(void);
chtype termattrs(void);
char *termname(void);
```

## termcap

```c
#include <curses.h>
#include <term.h>

extern char PC;
extern char * UP;
extern char * BC;
extern short ospeed;

int tgetent(char *bp, const char *name);
int tgetflag(const char *id);
int tgetnum(const char *id);
char *tgetstr(const char *id, char **area);
char *tgoto(const char *cap, int col, int row);
int tputs(const char *str, int affcnt, int (*putc)(int));
```
这些例程是作为使用termcap库的程序的转换辅助而包含的。它们的参数是相同的，但是这些例程是使用terminfo数据库模拟的。因此，它们只能用于查询已经编译了terminfo条目的条目的功能。

## terminfo

```c
#include <curses.h>
#include <term.h>

TERMINAL *cur_term;

const char * const boolnames[];
const char * const boolcodes[];
const char * const boolfnames[];
const char * const numnames[];
const char * const numcodes[];
const char * const numfnames[];
const char * const strnames[];
const char * const strcodes[];
const char * const strfnames[];

int setupterm(const char *term, int filedes, int *errret);
TERMINAL *set_curterm(TERMINAL *nterm);
int del_curterm(TERMINAL *oterm);
int restartterm(const char *term, int filedes, int *errret);

char *tparm(const char *str, ...);
int tputs(const char *str, int affcnt, int (*putc)(int));
int putp(const char *str);

int vidputs(chtype attrs, int (*putc)(int));
int vidattr(chtype attrs);
int vid_puts(attr_t attrs, short pair, void *opts, int (*putc)(int));
int vid_attr(attr_t attrs, short pair, void *opts);

int mvcur(int oldrow, int oldcol, int newrow, int newcol);

int tigetflag(const char *capname);
int tigetnum(const char *capname);
char *tigetstr(const char *capname);

char *tiparm(const char *str, ...);
```

## threads

```c
#include <curses.h>

typedef int (*NCURSES_WINDOW_CB)(WINDOW *, void *);
typedef int (*NCURSES_SCREEN_CB)(SCREEN *, void *);

int get_escdelay(void);
int set_escdelay(int ms);
int set_tabsize(int cols);

int use_screen(SCREEN *scr, NCURSES_SCREEN_CB func, void *data);
int use_window(WINDOW *win, NCURSES_WINDOW_CB func, void *data);
```

## touch 

```c
#include <curses.h>

int touchline(WINDOW *win, int start, int count);

int touchwin(WINDOW *win);
int wtouchln(WINDOW *win, int y, int n, int changed);

int untouchwin(WINDOW *win);

bool is_linetouched(WINDOW *win, int line);
bool is_wintouched(WINDOW *win);
```

## trace

```c
#include <curses.h>

unsigned curses_trace(const unsigned param);

void _tracef(const char *format, ...);

char *_traceattr(attr_t attr);
char *_traceattr2(int buffer, chtype ch);
char *_tracecchar_t(const cchar_t *string);
char *_tracecchar_t2(int buffer, const cchar_t *string);
char *_tracechar(int ch);
char *_tracechtype(chtype ch);
char *_tracechtype2(int buffer, chtype ch);

void _tracedump(const char *label, WINDOW *win);
char *_nc_tracebits(void);
char *_tracemouse(const MEVENT *event);

/* deprecated */
void trace(const unsigned int param);
```

## util

```c
#include <curses.h>

const char *unctrl(chtype c);
wchar_t *wunctrl(cchar_t *c);

const char *keyname(int c);
const char *key_name(wchar_t w);

void filter(void);
void nofilter(void);

void use_env(bool f);
void use_tioctl(bool f);

int putwin(WINDOW *win, FILE *filep);
WINDOW *getwin(FILE *filep);

int delay_output(int ms);
int flushinp(void);
```

## variables

```c
#include <curses.h>

int COLOR_PAIRS;
int COLORS;
int COLS;
int ESCDELAY;
int LINES;
int TABSIZE;
WINDOW* curscr;
WINDOW* newscr;
WINDOW* stdscr;
```

## window

```c
#include <curses.h>

WINDOW *newwin(int nlines, int ncols, int begin_y, int begin_x);
int delwin(WINDOW *win);
int mvwin(WINDOW *win, int y, int x);
WINDOW *subwin(WINDOW *orig, int nlines, int ncols, int begin_y, int begin_x);
WINDOW *derwin(WINDOW *orig, int nlines, int ncols, int begin_y, int begin_x);
int mvderwin(WINDOW *win, int par_y, int par_x);
WINDOW *dupwin(WINDOW *win);
void wsyncup(WINDOW *win);
int syncok(WINDOW *win, bool bf);
void wcursyncup(WINDOW *win);
void wsyncdown(WINDOW *win);
```

## default_color

```c
#include <curses.h>

int use_default_colors(void);
int assume_default_colors(int fg, int bg);
```

## define_key

```c
#include <curses.h>

int define_key(const char *definition, int keycode);
```

## form_cursor

```c
#include <form.h>

int pos_form_cursor(FORM *form);
```

## form_data

```c
#include <form.h>

bool data_ahead(const FORM *form);
bool data_behind(const FORM *form);
```

## form_driver

```c
#include <form.h>

int form_driver(FORM *form, int c);
int form_driver_w(FORM *form, int c, wchar_t wch);
```

## form_field_attributes

```c
#include <form.h>

int set_field_fore(FIELD *field, chtype attr);
chtype field_fore(const FIELD *field);

int set_field_back(FIELD *field, chtype attr);
chtype field_back(const FIELD *field);

int set_field_pad(FIELD *field, int pad);
int field_pad(const FIELD *field);
```

## form_field_buffer

```c
#include <form.h>

int set_field_buffer(FIELD *field, int buf, const char *value);
char *field_buffer(const FIELD *field, int buffer);

int set_field_status(FIELD *field, bool status);
bool field_status(const FIELD *field);

int set_max_field(FIELD *field, int max);
```

## form_field_info

```c
#include <form.h>

int field_info(const FIELD *field, int *rows, int *cols, int *frow, int *fcol, int *nrow, int *nbuf);

int dynamic_field_info(const FIELD *field, int *rows, int *cols, int *max);
```

## form_field_just

```c
#include <form.h>

int set_field_just(FIELD *field, int justification);
int field_just(const FIELD *field);
```

## form_field_new

```c
#include <form.h>

FIELD *new_field(int height, int width, int toprow, int leftcol, int offscreen, int nbuffers);
FIELD *dup_field(FIELD *field, int toprow, int leftcol);
FIELD *link_field(FIELD *field, int toprow, int leftcol);
int free_field(FIELD *field);
```

## form_field_opts

```c
#include <form.h>

int set_field_opts(FIELD *field, Field_Options opts);
Field_Options field_opts(const FIELD *field);

int field_opts_on(FIELD *field, Field_Options opts);
int field_opts_off(FIELD *field, Field_Options opts);
```

## form_field_userptr

```c
#include <form.h>

int set_field_userptr(FIELD *field, void *userptr);
void *field_userptr(const FIELD *field);
```

## form_field_validation

```c
#include <form.h>

void *field_arg(const FIELD *field);
FIELDTYPE *field_type(const FIELD *field);
int set_field_type(FIELD *field, FIELDTYPE *type, ...);

/* predefined field types */
FIELDTYPE *TYPE_ALNUM;
FIELDTYPE *TYPE_ALPHA;
FIELDTYPE *TYPE_ENUM;
FIELDTYPE *TYPE_INTEGER;
FIELDTYPE *TYPE_NUMERIC;
FIELDTYPE *TYPE_REGEXP;
FIELDTYPE *TYPE_IPV4;
```

## form_field

```c
#include <form.h>

int set_form_fields(FORM *form, FIELD **fields);
FIELD **form_fields(const FORM *form);
int field_count(const FORM *form);
int move_field(FIELD *field, int frow, int fcol);
```

## form_fieldtype

```c
#include <form.h>

FIELDTYPE *new_fieldtype(bool (* const field_check)(FIELD *, const void *), bool (* const char_check)(int, const void *));
int free_fieldtype(FIELDTYPE *fieldtype);

int set_fieldtype_arg(FIELDTYPE *fieldtype, void *(* const make_arg)(va_list *), void *(* const copy_arg)(const void *), void  (* const free_arg)(void *));
int set_fieldtype_choice(FIELDTYPE *fieldtype, bool (* const next_choice)(FIELD *, const void *), bool (* const prev_choice)(FIELD *, const void *));

FIELDTYPE *link_fieldtype(FIELDTYPE *type1, FIELDTYPE *type2);
```

## form_hook

```c
#include <form.h>

int set_field_init(FORM *form, Form_Hook func);
Form_Hook field_init(const FORM *form);

int set_field_term(FORM *form, Form_Hook func);
Form_Hook field_term(const FORM *form);

int set_form_init(FORM *form, Form_Hook func);
Form_Hook form_init(const FORM *form);

int set_form_term(FORM *form, Form_Hook func);
Form_Hook form_term(const FORM *form);
```

## form_new_page

```c
#include <form.h>

int set_new_page(FIELD *field, bool new_page_flag);
bool new_page(const FIELD *field);
```

## form_new

```c
#include <form.h>

FORM *new_form(FIELD **fields);
int free_form(FORM *form);
```

## form_opts

```c
#include <form.h>

int set_form_opts(FORM *form, Field_Options opts);
Field_Options form_opts(const FORM *form);

int form_opts_on(FORM *form, Field_Options opts);
int form_opts_off(FORM *form, Field_Options opts);
```

## form_page

```c
#include <form.h>

int set_current_field(FORM *form, FIELD *field);
FIELD *current_field(const FORM *form);

int unfocus_current_field(FORM *form);

int set_form_page(FORM *form, int n);
int form_page(const FORM *form);

int field_index(const FIELD *field);
```

## form_post

```c
#include <form.h>

int post_form(FORM *form);
int unpost_form(FORM *form);
```

## form_requestname

```c
#include <form.h>

const char *form_request_name(int request);
int form_request_by_name(const char *name);
```

## form_userptr

```c
#include <form.h>

int set_form_userptr(FORM *form, void *userptr);
void* form_userptr(const FORM *form);
```

## form_variables

```c
#include <form.h>

FIELDTYPE* TYPE_ALNUM;
FIELDTYPE* TYPE_ALPHA;
FIELDTYPE* TYPE_ENUM;
FIELDTYPE* TYPE_INTEGER;
FIELDTYPE* TYPE_IPV4;
FIELDTYPE* TYPE_NUMERIC;
FIELDTYPE* TYPE_REGEXP;
```

## form_win

```c
#include <form.h>

int set_form_win(FORM *form, WINDOW *win);
WINDOW *form_win(const FORM *form);

int set_form_sub(FORM *form, WINDOW *sub);
WINDOW *form_sub(const FORM *form);

int scale_form(const FORM *form, int *rows, int *columns);
```

## key_Defined

```c
#include <curses.h>

int key_defined(const char *definition);
```

## keybound

```c
#include <curses.h>

char* keybound(int keycode, int count);
```

## keyok

```c
#include <curses.h>

int keyok(int keycode, bool enable);
```

## legacy_coding

```c
#include <curses.h>

int use_legacy_coding(int level);
```

## menu_attributes

```c
#include <menu.h>

int set_menu_fore(MENU *menu, chtype attr);
chtype menu_fore(const MENU *menu);

int set_menu_back(MENU *menu, chtype attr);
chtype menu_back(const MENU *menu);

int set_menu_grey(MENU *menu, chtype attr);
chtype menu_grey(const MENU *menu);

int set_menu_pad(MENU *menu, int pad);
int menu_pad(const MENU *menu);
```

## menu_cursor

```c
#include <menu.h>

int pos_menu_cursor(const MENU *menu);
```

## menu_driver

```c
#include <menu.h>

int menu_driver(MENU *menu, int c);
```

## menu_format

```c
#include <menu.h>

int set_menu_format(MENU *menu, int rows, int cols);
void menu_format(const MENU *menu, int *rows, int *cols);
```

## menu_hook

```c
#include <menu.h>

int set_item_init(MENU *menu, Menu_Hook func);
Menu_Hook item_init(const MENU *menu);

int set_item_term(MENU *menu, Menu_Hook func);
Menu_Hook item_term(const MENU *menu);

int set_menu_init(MENU *menu, Menu_Hook func);
Menu_Hook menu_init(const MENU *menu);

int set_menu_term(MENU *menu, Menu_Hook func);
Menu_Hook menu_term(const MENU *menu);
```

## menu_items

```c
#include <menu.h>

int set_menu_items(MENU *menu, ITEM **items);
ITEM **menu_items(const MENU *menu);
int item_count(const MENU *menu);
```

## menu_mark

```c
#include <menu.h>

int set_menu_mark(MENU *menu, const char *mark);
const char *menu_mark(const MENU *menu);
```

## menu_new

```c
#include <menu.h>

MENU *new_menu(ITEM **items);
int free_menu(MENU *menu);
```

## menu_opts

```c
#include <menu.h>

int set_menu_opts(MENU *menu, Menu_Options opts);
Menu_Options menu_opts(const MENU *menu);

int menu_opts_on(MENU *menu, Menu_Options opts);
int menu_opts_off(MENU *menu, Menu_Options opts);
```

## menu_pattern

```c
#include <menu.h>

int set_menu_pattern(MENU *menu, const char *pattern);
char *menu_pattern(const MENU *menu);
```

## menu_post

```c
#include <menu.h>

int post_menu(MENU *menu);
int unpost_menu(MENU *menu);
```

## menu_requestname

```c
#include <menu.h>

const char *menu_request_name(int request);
int menu_request_by_name(const char *name);
```

## menu_spacing

```c
#include <menu.h>

int set_menu_spacing(MENU *menu, int spc_description, int spc_rows, int spc_columns);
int menu_spacing(const MENU *menu, int* spc_description, int* spc_rows, int* spc_columns);
```

## menu_userptr

```c
#include <menu.h>

int set_menu_userptr(MENU *menu, void *userptr);
void *menu_userptr(const MENU *menu);
```

## menu_win

```c
#include <menu.h>

int set_menu_win(MENU *menu, WINDOW *win);
WINDOW *menu_win(const MENU *menu);

int set_menu_sub(MENU *menu, WINDOW *sub);
WINDOW *menu_sub(const MENU *menu);

int scale_menu(const MENU *menu, int *rows, int *columns);
```

## mitem_current

```c
#include <menu.h>

int set_current_item(MENU *menu, ITEM *item);
ITEM *current_item(const MENU *menu);

int set_top_row(MENU *menu, int row);
int top_row(const MENU *menu);

int item_index(const ITEM *item);
```

## mitem_name

```c
#include <menu.h>

const char *item_name(const ITEM *item);
const char *item_description(const ITEM *item);
```

## mitem_new

```c
#include <menu.h>

ITEM *new_item(const char *name, const char *description);
int free_item(ITEM *item);
```

## mitem_opts

```c
#include <menu.h>

int set_item_opts(ITEM *item, Item_Options opts);
Item_Options item_opts(const ITEM *item);

int item_opts_on(ITEM *item, Item_Options opts);
int item_opts_off(ITEM *item, Item_Options opts);
```

## mitem_userptr

```c
#include <menu.h>

int set_item_userptr(ITEM *item, void *userptr);
void *item_userptr(const ITEM *item);
```

## mitem_value

```c
#include <menu.h>

int set_item_value(ITEM *item, bool value);
bool item_value(const ITEM *item);
```

## mitem_visible

```c
 #include <menu.h>

bool item_visible(const ITEM *item);
```

## new_pair

```c
#include <curses.h>

int alloc_pair(int fg, int bg);
int find_pair(int fg, int bg);
int free_pair(int pair);
```

## panel

```c
#include <panel.h>

cc [flags] sourcefiles -lpanel -lncurses

PANEL *new_panel(WINDOW *win);

int bottom_panel(PANEL *pan);
int top_panel(PANEL *pan);
int show_panel(PANEL *pan);
void update_panels(void);
int hide_panel(PANEL *pan);

WINDOW *panel_window(const PANEL *pan);
int replace_panel(PANEL *pan, WINDOW *window);
int move_panel(PANEL *pan, int starty, int startx);
int panel_hidden(const PANEL *pan);

PANEL *panel_above(const PANEL *pan);
PANEL *panel_below(const PANEL *pan);

int set_panel_userptr(PANEL *pan, const void *ptr);
const void *panel_userptr(const PANEL *pan);

int del_panel(PANEL *pan);

/* ncurses-extensions */
PANEL *ground_panel(SCREEN *sp);
PANEL *ceiling_panel(SCREEN *sp);
```

## resizeterm

```c
#include <curses.h>

bool is_term_resized(int lines, int columns);
int resize_term(int lines, int columns);
int resizeterm(int lines, int columns);
```

## term_variables

```c
#include <curses.h>
#include <term.h>

chtype acs_map[];

SCREEN * SP;

TERMINAL * cur_term;

char ttytype[];

NCURSES_CONST char * const boolcodes[];
NCURSES_CONST char * const boolfnames[];
NCURSES_CONST char * const boolnames[];

NCURSES_CONST char * const numcodes[];
NCURSES_CONST char * const numfnames[];
NCURSES_CONST char * const numnames[];

NCURSES_CONST char * const strcodes[];
NCURSES_CONST char * const strfnames[];
NCURSES_CONST char * const strnames[];
```

## wresize

```c
#include <curses.h>

int wresize(WINDOW *win, int lines, int columns);
```
