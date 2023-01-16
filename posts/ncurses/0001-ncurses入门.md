---
title: "ncurses入门"
date: 2023-01-11T21:03:48+08:00
tags: ['ncurses', 'linux']
categories: ['ncurses']
draft: false
---

## 简介

在电传终端的旧时代，终端远离计算机，通过串行电缆与计算机连接。终端可以通过发送一系列字节来配置。终端的所有功能(如移动光标到一个新的位置，擦除屏幕的一部分，滚动屏幕，改变模式等)都可以通过这一系列字节访问。这些控制序列通常被称为转义序列，因为它们以一个转义(0x1B)字符开始。即使在今天，通过适当的模拟，我们也可以向模拟器发送转义序列，并在终端窗口上实现相同的效果。

假设你想用彩色打印一行。试着在控制台上输入如下代码：
```shell
echo "^[[0;31;40mIn Color"
```

第一个字符是转义字符，看起来像两个字符`^`和`[`。要打印它，你必须按CTRL+V，然后按ESC键。所有其他字符都是正常的可打印字符。你应该可以看到字符串“In Color”是红色的。它保持这种方式，并恢复到原始模式键入这个。
```shell
echo "^[[0;37;40m"
```

现在，这些神奇的字符是什么意思?难以理解?对于不同的终端，它们甚至可能是不同的。

因此UNIX的设计者发明了一种名为`termcap`的机制。它是一个列出特定终端的所有功能的文件，以及实现特定效果所需的转义序列。

在后来的几年里，这被`terminfo`所取代。这种机制允许应用程序查询`terminfo`数据库并获得要发送到终端或终端模拟器的控制字符，而无需深入研究太多细节。

### ncurses 是什么?
在上面的场景中，每个应用程序都应该查询terminfo并执行必要的操作(发送控制字符等)。

很快就很难管理这种复杂性，这就产生了“CURSES”。

Curses是“游标优化”名称的双关语。

Curses库在使用原始终端代码时形成了一个包装器，并提供了高度灵活和高效的API(应用程序编程接口)。

它提供了移动光标，创建窗口，产生颜色，玩鼠标等功能。应用程序不需要担心底层终端功能。

> 什么是NCURSES?<br/>
> NCURSES是原System V Release 4.0 (SVr4) curses的克隆。
> 它是一个可自由分发的库，完全兼容旧版本的curses。
> <br/>简而言之，它是一个函数库，用于管理应用程序在字符单元终端上的显示。在该文件的其余部分，术语curses和ncurses互换使用。

### 使用ncurses可以做什么?

NCURSES不仅在终端功能上创建了一个包装器，而且还提供了一个健壮的框架，以文本模式创建漂亮的UI(用户界面)。

它提供了创建窗口等功能。它的姐妹库 `panel`，`menu` 和`forms`提供了一个扩展库。可以创建包含多个窗口、菜单、面板和表单的应用程序。窗口可以独立管理，可以提供“可滚动性”，甚至可以隐藏。

菜单为用户提供了一个简单的命令选择选项。表单允许创建易于使用的数据输入和显示窗口。面板扩展了ncurses处理重叠和堆叠窗口的能力。

这些只是我们可以用ncurses做的一些基本事情。随着我们的前进，我们将看到这些库的所有功能。

### 项目结构
```
ncurses
   |
   |----> JustForFun     -- just for fun programs
   |----> basics         -- basic programs
   |----> demo           -- output files go into this directory after make
   |          |
   |          |----> exe -- exe files of all example programs
   |----> forms          -- programs related to form library
   |----> menus          -- programs related to menus library
   |----> panels         -- programs related to panels library
   |----> perl           -- perl equivalents of the examples (contributed
   |                            by Anuradha Ratnaweera)
   |----> Makefile       -- the top level Makefile
   |----> README         -- the top level README file. contains instructions
   |----> COPYING        -- copyright notice
```

文件详细信息
```
Description of files in each directory
--------------------------------------
JustForFun
    |
    |----> hanoi.c   -- The Towers of Hanoi Solver
    |----> life.c    -- The Game of Life demo
    |----> magic.c   -- An Odd Order Magic Square builder 
    |----> queens.c  -- The famous N-Queens Solver
    |----> shuffle.c -- A fun game, if you have time to kill
    |----> tt.c      -- A very trivial typing tutor

  basics
    |
    |----> acs_vars.c            -- ACS_ variables example
    |----> hello_world.c         -- Simple "Hello World" Program
    |----> init_func_example.c   -- Initialization functions example
    |----> key_code.c            -- Shows the scan code of the key pressed
    |----> mouse_menu.c          -- A menu accessible by mouse
    |----> other_border.c        -- Shows usage of other border functions apa
    |                               -- rt from box()
    |----> printw_example.c      -- A very simple printw() example
    |----> scanw_example.c       -- A very simple getstr() example
    |----> simple_attr.c         -- A program that can print a c file with 
    |                               -- comments in attribute
    |----> simple_color.c        -- A simple example demonstrating colors
    |----> simple_key.c          -- A menu accessible with keyboard UP, DOWN 
    |                               -- arrows
    |----> temp_leave.c          -- Demonstrates temporarily leaving curses mode
    |----> win_border.c          -- Shows Creation of windows and borders
    |----> with_chgat.c          -- chgat() usage example

  forms 
    |
    |----> form_attrib.c     -- Usage of field attributes
    |----> form_options.c    -- Usage of field options
    |----> form_simple.c     -- A simple form example
    |----> form_win.c        -- Demo of windows associated with forms

  menus 
    |
    |----> menu_attrib.c     -- Usage of menu attributes
    |----> menu_item_data.c  -- Usage of item_name() etc.. functions
    |----> menu_multi_column.c    -- Creates multi columnar menus
    |----> menu_scroll.c     -- Demonstrates scrolling capability of menus
    |----> menu_simple.c     -- A simple menu accessed by arrow keys
    |----> menu_toggle.c     -- Creates multi valued menus and explains
    |                           -- REQ_TOGGLE_ITEM
    |----> menu_userptr.c    -- Usage of user pointer
    |----> menu_win.c        -- Demo of windows associated with menus

  panels 
    |
    |----> panel_browse.c    -- Panel browsing through tab. Usage of user 
    |                           -- pointer
    |----> panel_hide.c      -- Hiding and Un hiding of panels
    |----> panel_resize.c    -- Moving and resizing of panels
    |----> panel_simple.c    -- A simple panel example

  perl
    |----> 01-10.pl          -- Perl equivalents of first ten example programs
```

## Hello World

```c
#include <ncurses.h>

int main()
{	
	initscr();			/* Start curses mode 		  */
	printw("Hello World !!!");	/* Print Hello World		  */
	refresh();			/* Print it on to the real screen */
	getch();			/* Wait for user input */
	endwin();			/* End curses mode		  */

	return 0;
}
```

### 代码分析

#### initscr()

函数`initscr()`以curses模式初始化终端。

在某些实现中，它会清除屏幕并显示空白屏幕。

要做任何屏幕操作使用ncurses这必须首先被调用。

这个函数初始化curses系统，并为当前窗口(称为stdscr)和其他一些数据结构分配内存。

在极端情况下，由于没有足够的内存为curses库的数据结构分配内存，该函数可能会失败。

完成此操作后，我们可以进行各种初始化以自定义curses设置。这些细节将在后面解释。

#### refresh()

下一行printw将字符串“Hello World !!”打印到屏幕上。

这个函数在各个方面都类似于普通的printf，除了它在一个名为stdscr的窗口上打印当前(y,x)坐标上的数据。

由于我们当前的坐标是0,0，字符串被打印在窗口的左上角。

这就引出了神秘的refresh()。

当我们调用printw时，数据实际上被写入了一个假想的窗口，这个窗口还没有在屏幕上更新。

printw的任务是更新一些标志和数据结构，并将数据写入与stdscr对应的缓冲区。

为了在屏幕上显示它，我们需要调用refresh()并告诉curses系统将内容转储到屏幕上。

这一切背后的理念是允许程序员在想象的屏幕或窗口上进行多次更新，并在所有屏幕更新完成后进行刷新。

refresh()检查窗口并仅更新已更改的部分。这提高了性能并提供了更大的灵活性。

但是，初学者有时会感到沮丧。初学者常犯的一个错误是在通过printw()类函数进行更新后忘记调用refresh()。我有时还是会忘记加上它:-)

#### endwin()

最后别忘了结束ncurses模式。否则，在程序退出后，您的终端可能会表现异常。

endwin()释放curses子系统及其数据结构占用的内存，并将终端置于正常模式。此函数必须在使用curses模式后调用。


## ncurses 初始化

现在我们知道，要初始化curses系统，必须调用函数initscr()。在初始化之后，可以调用一些函数来定制curses会话。我们可以要求curses系统将终端设置为原始模式或初始化颜色或初始化鼠标等。让我们讨论一些通常在initscr()之后立即调用的函数;

### raw() 和 cbreak()

通常，终端驱动程序会缓冲用户输入的字符，直到遇到新的行或回车。但是大多数程序都要求用户在输入字符时就可以使用这些字符。**以上两个功能用于禁用行缓冲**。

这两个函数之间的区别在于传递给程序的控制字符，如暂停(CTRL-Z)、中断和退出(CTRL-C)。

在raw()模式下，这些字符直接传递给程序而不生成信号。

在cbreak()模式下，终端驱动程序将这些控制字符解释为任何其他字符。

我个人更喜欢使用raw()，因为我可以更好地控制用户的操作。

### echo() 和 noecho()

这些函数控制用户输入的字符到终端的回显。

noecho()关闭回声，这样做的原因可能是为了获得对回显的更多控制，或者在通过getch()等函数从用户获取输入时抑制不必要的回显。

大多数交互式程序在初始化时调用noecho()，并以受控的方式进行字符回显。它使程序员能够灵活地在窗口的任何位置回显字符，而无需更新当前(y,x)坐标。

### keypad()

这是我最喜欢的初始化函数。它可以读取功能键，如F1, F2，方向键等。

几乎每个交互程序都支持这一点，因为方向键是任何用户界面的主要部分。

执行 `keypad(stdscr, TRUE)` 在常规屏幕(stdscr)上启用此功能。在本文档后面的部分中，您将了解有关按键管理的更多信息。

### halfdelay()

这个函数虽然不常用，但有时很有用。调用halfdelay()来启用半延迟模式，它类似于cbreak()模式，输入的字符立即可用于程序。

但是，如果没有可用的输入，它会等待'X'十分之一秒来输入，然后返回ERR。'X'是传递给函数halfdelay()的超时值。

当你想让用户输入，如果他在一定时间内没有回应，这个函数是有用的，我们可以做一些其他的事情。一个可能的例子是密码提示处的超时。

### getch()

函数getch()用于从user获取一个字符。它相当于普通的getchar()。

## Windows 简介

在我们深入研究无数ncurses函数之前，让我先澄清一些关于窗口的事情。下面几节将详细解释Windows。

窗口是由curses系统定义的假想屏幕。窗口并不意味着通常在Win9X平台上看到的有边框的窗口。

当curses初始化时，它会创建一个名为`stdscr的默认窗口`，该窗口表示您的80x25(或您正在运行的窗口的大小)屏幕。

如果你正在做一些简单的任务，比如打印一些字符串，读取输入等，你可以安全地使用这个窗口来完成所有的任务。您还可以创建窗口并调用显式工作在指定窗口上的函数。

```c
printw("Hi There !!!");
refresh();
```
它打印stdscr上当前光标位置的字符串。类似地，对refresh()的调用仅适用于stdscr。


假设你已经创建了窗口，那么你必须调用一个函数，在通常的函数中添加一个'w'。
```c
wprintw(win, "Hi There !!!");
wrefresh(win);
```
正如您将在文档的其余部分看到的，函数的命名遵循相同的约定。每个函数通常有三个以上的函数。

```c
printw(string);        /* Print on stdscr at present cursor position */
mvprintw(y, x, string);/* Move to (y, x) then print string     */
wprintw(win, string);  /* Print on window win at present cursor position */
                       /* in the window */
mvwprintw(win, y, x, string);   /* Move to (y, x) relative to window */
                                /* co-ordinates and then print         */
```

通常w-less函数是宏，它以stdscr作为窗口参数展开为相应的w-function。

## 输出函数

至此ncurses初始化完成，接下来让我们完成一些交互工作。

你可以使用三类函数在屏幕上进行输出。

- `addch()`：打印带有属性的单个字符
- `printw()`：类似printf()一样的格式化输出
- `addstr()`：打印字符串

这些函数可以互换使用，使用哪个类只是风格问题。让我们详细看看每一个。

### addch()

这些函数将单个字符放入当前光标位置，并向前移动光标的位置。您可以指定要打印的字符，但它们通常用于打印具有某些属性的字符。属性将在文档后面的部分中详细解释。如果一个字符与一个属性相关联(粗体，反向视频等)，当curses打印该字符时，它将被打印在该属性中。

为了将一个字符与某些属性结合起来，你有两个选择:
- 通过将单个字符与所需的属性宏进行OR运算。这些属性宏可以在头文件ncurses.h中找到。例如，你想打印一个字符ch(char类型)加粗加下划线，你可以调用addch()，如下所示。
    ```c
    addch(ch | A_BOLD | A_UNDERLINE);
    ```
- 通过使用`attrset()`、`attron()`、`attrff()`等函数。属性部分解释了这些函数。简单地说，它们操作给定窗口的当前属性。设置完成后，打印在窗口中的字符将与属性相关联，直到将其关闭。

此外，curses为基于字符的图形提供了一些特殊字符。你可以画表格、水平线或垂直线等。您可以在头文件ncurses.h中找到所有可用的字符。尝试在这个文件中寻找以ACS_开头的宏。

### mvaddch(), waddch()和mvwaddch()

`mvaddch()`用于将光标移动到给定点，然后打印。例子:
```c
move(row,col);    /* moves the cursor to rowth row and colth column */
addch(ch);
```

可以被替换为：

```c
mvaddch(row, col, ch);
```
Waddch()与addch()类似，只是它将一个字符添加到给定的窗口中。(注意addch()将一个字符添加到窗口stdscr中。)

以类似的方式使用mvwaddch()函数将字符添加到给定坐标的给定窗口中。

现在，我们已经熟悉了基本的输出函数addch()。但是，如果我们想打印一个字符串，一个字符一个字符地打印是很烦人的。幸运的是，ncurses提供了类似printf或类似put的函数。

### printw()

这些函数类似于printf()，只不过增加了在屏幕上任何位置打印的功能。

#### printw()和mvprintw()

这两个函数的工作原理很像printf()。Mvprintw()可以用来移动光标到一个位置，然后打印。如果你想先移动光标，然后使用printw()函数打印，先使用move()，然后使用printw()，尽管我不明白为什么应该避免使用mvprintw()，你可以灵活操作。

#### wprintw()和mvwprintw()

这两个函数与上面两个函数相似，除了它们打印在作为参数给定的相应窗口中。

#### vwprintw()

这个函数类似于vprintf()。当要打印可变数量的参数时，可以使用此方法。

#### 例子

```c
#include <ncurses.h>            /* ncurses.h includes stdio.h */  
#include <string.h> 

int main()
{
    char mesg[]="Just a string";    /* message to be appeared on the screen */
    int row, col;                   /* to store the number of rows and *
                                     * the number of colums of the screen */
    initscr();                      /* start the curses mode */
    getmaxyx(stdscr, row, col);     /* get the number of rows and columns */
    mvprintw(row / 2, (col-strlen(mesg)) / 2, "%s", mesg);

    /* print the message at the center of the screen */
    mvprintw(row-2, 0, "This screen has %d rows and %d columns\n", row, col);
    printw("Try resizing your window(if possible) and then run this program again");

    refresh();
    getch();

    endwin();

    return 0;
}
```

上面的程序演示了使用printw是多么容易。您只需输入坐标和要显示在屏幕上的消息，然后它就会执行您想要的操作。

上面的程序向我们介绍了一个新函数getmaxyx()，它是在ncurses.h中定义的宏。它给出了给定窗口中的列数和行数。

getmaxyx()通过更新给定的变量来实现这一点。因为getmaxyx()不是一个函数，我们不向它传递指针，我们只给出两个整数变量。

### addstr()

addstr()用于将字符串放入给定的窗口中。这个函数类似于为给定字符串中的每个字符调用一次addch()。对于所有输出函数都是如此。这个家族中还有其他函数，如mvaddstr()，mvwaddstr()和waddstr()，它们遵循curses的命名约定。mvaddstr()类似于分别调用move()和addstr()。这个家族的另一个函数是addnstr()，它额外接受一个整数形参(比如n)。这个函数最多在屏幕上放置n个字符。如果n为负，则整个字符串将被添加。

> 注意：
> 所有这些函数的参数都先取y坐标，然后取x坐标。
>
> 初学者常犯的一个错误是按这个顺序传递x和y。如果你对(y,x)坐标做了太多的操作，考虑把屏幕分成几个窗口，然后分别操作每个窗口。窗口将在窗口一节中进行解释。

## 输入函数

没有输入的打印是很无聊的。让我们看看允许我们从用户那里获取输入的函数。这些功能也可以分为三类。
- getch()：输入一个字符
- scanw()：格式化输入
- getstr()：输入一个字符串

### getch()
这些函数从终端读取单个字符。

但有几个微妙的事实需要考虑。例如，如果不使用函数cbreak()，curses将不会连续读取输入字符，而是在遇到新行或EOF后才开始读取它们。为了避免这种情况，必须使用cbreak()函数，以便程序可以立即使用字符。

另一个广泛使用的函数是noecho()。顾名思义，当这个函数被设置(使用)时，用户输入的字符将不会显示在屏幕上。两个函数cbreak()和noecho()是密钥管理的典型示例。密钥管理部分解释了这种类型的功能。

### scanw()

这些函数类似于scanf()，只不过增加了从屏幕上任何位置获取输入的功能。

#### scanw() 和 mvscanw()

这些函数的用法类似于sscanf()，其中要扫描的行是由wgetstr()函数提供的。也就是说，这些函数调用wgetstr()函数(下面解释)并使用结果行进行扫描。

#### wscanw() 和 mvwscanw()

这些函数类似于上面两个函数，除了它们从窗口读取，而窗口是作为这些函数的参数之一提供的。

#### vwscanw()

这个函数类似于vscanf()。当要扫描的参数数量可变时，可以使用这种方法。

### getstr()

这些函数用于从终端获取字符串。本质上，这个函数执行的任务与一系列调用`getch()`所实现的任务相同，直到接收到换行符、回车符或文件结束符。生成的字符字符串由str指向，str是用户提供的字符指针。

### 例子

```c
#include <ncurses.h>            /* ncurses.h includes stdio.h */  
#include <string.h> 

int main()
{
    char mesg[]="Just a string";    /* message to be appeared on the screen */
    int row, col;                   /* to store the number of rows and *
                                     * the number of colums of the screen */
    initscr();                      /* start the curses mode */
    getmaxyx(stdscr, row, col);     /* get the number of rows and columns */
    mvprintw(row / 2, (col-strlen(mesg)) / 2, "%s", mesg);

    /* print the message at the center of the screen */
    mvprintw(row-2, 0, "This screen has %d rows and %d columns\n", row, col);
    printw("Try resizing your window(if possible) and then run this program again");

    refresh();
    getch();

    endwin();

    return 0;
}
```

## 属性

我们已经看到了如何使用属性打印具有某些特殊效果的字符的示例。如果谨慎地设置属性，可以以简单、可理解的方式显示信息。下面的程序以一个C文件作为输入，并打印带有粗体注释的文件。扫描代码。

例子
```c
#include <ncurses.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{ 
    int ch, prev, row, col;
    prev = EOF;
    FILE *fp;
    int y, x;

    if(argc != 2) {
        printf("Usage: %s <a c file name>\n", argv[0]);
        exit(1);
    }

    fp = fopen(argv[1], "r");
    if(fp == NULL) {
        perror("Cannot open input file");
        exit(1);
    }

    initscr();                              /* Start curses mode */

    getmaxyx(stdscr, row, col);             /* find the boundaries of the screeen */
    while ((ch = fgetc(fp)) != EOF) {       /* read the file till we reach the end */
        getyx(stdscr, y, x);                /* get the current curser position */
        if(y == (row - 1)) {                /* are we are at the end of the screen */
            printw("<-Press Any Key->");    /* tell the user to press a key */
            getch();
            clear();                        /* clear the screen */
            move(0, 0);                     /* start at the beginning of the screen */
        }

        if(prev == '/' && ch == '*') {      /* If it is / and * then only switch bold on */    
            attron(A_BOLD);                 /* cut bold on */
            getyx(stdscr, y, x);            /* get the current curser position */
            move(y, x - 1);                 /* back up one space */
            printw("%c%c", '/', ch);        /* The actual printing is done here */
        }
        else {
            printw("%c", ch);
        }

        refresh();
        if(prev == '*' && ch == '/') {
            attroff(A_BOLD);                /* Switch it off once we got and then */
        }

        prev = ch;
    }

    endwin();                               /* End curses mode */

    fclose(fp);

    return 0;
}
```
不要担心所有那些初始化和其他废话。专注于while循环。它读取文件中的每个字符并搜索模式`/*`。一旦它发现了模式，就会使用attron()打开BOLD属性。当我们得到模式`*/`时，它被attroff()关闭。

上面的程序还介绍了两个有用的函数getyx()和move()。第一个函数将当前光标的坐标转换为变量y, x。由于getyx()是一个宏，我们不必将指针传递给变量。函数move()将光标移动到给定的坐标。

上面的程序实际上是一个简单的程序，它没有做很多事情。在这些行中，我们可以编写一个更有用的程序，它可以读取C文件，解析它，并以不同的颜色打印它。人们甚至可以将其扩展到其他语言。

### 属性细节

让我们更详细地了解属性。函数`attron()`、`attroff()`、`attrset()`以及它们的姐妹函数`attr_get()`等。可用于打开/关闭属性，获取属性并产生彩色显示。

函数attron和attroff接受属性的位掩码，并分别打开或关闭它们。下面定义在`<curses.h>`中的视频属性可以传递给这些函数。

```c
A_NORMAL        Normal display (no highlight)
A_STANDOUT      Best highlighting mode of the terminal.
A_UNDERLINE     Underlining
A_REVERSE       Reverse video
A_BLINK         Blinking
A_DIM           Half bright
A_BOLD          Extra bright or bold
A_PROTECT       Protected mode
A_INVIS         Invisible or blank mode
A_ALTCHARSET    Alternate character set
A_CHARTEXT      Bit-mask to extract a character
COLOR_PAIR(n)   Color-pair number n 
```

最后一个是最丰富多彩的一个:-)颜色将在下一节中解释。

我们可以OR(|)以上任意数量的属性来获得组合效果。如果你想用闪烁字符反向视频，你可以使用：
```c
attron(A_REVERSE | A_BLINK);
```

### attron()和attrset()

那么attron()和attrset()有什么区别呢?

attrset设置窗口的属性，而attron只是打开给定的属性。因此attrset()完全覆盖窗口之前拥有的任何属性，并将其设置为新的属性。

类似地，attroff()只是关闭了作为参数赋予它的属性。这为我们提供了轻松管理属性的灵活性。但是如果你不小心使用它们，你可能会丢失窗口的属性并混淆显示。在管理带有颜色和高亮显示的菜单时尤其如此。所以，制定一个一致的政策，并坚持下去。您总是可以使用standend()，它相当于`attrset(A_NORMAL)`，它会关闭所有属性并将您带到正常模式。

### attr\_get()

函数attr\_get()获取窗口的当前属性和颜色对。虽然我们可能不像上面的函数那样经常使用它，但它在扫描屏幕区域时很有用。假设我们想要在屏幕上进行一些复杂的更新，并且我们不确定每个字符与什么属性相关联。然后，该函数可以与attrset或attron一起使用，以产生所需的效果。

### attr\_functions
有一系列函数，如`attr_set()`， `attr_on()`等。这些函数与上面的函数类似，只是它们接受`attr_t`类型的参数。

### wattr\_functions

对于上面的每个函数，我们都有一个对应的带有“w”的函数，它对特定的窗口进行操作。上述函数对stdscr进行操作。


### chgat()

函数`chgat()`列在手册`curs_attr`的末尾。这实际上是一个有用的方法。此函数可用于设置一组字符的属性而不移动。我是认真的!-)它从当前光标位置开始改变给定数量的字符的属性。

我们可以给-1作为字符数来更新直到行结束。如果你想改变字符的属性从当前位置到行尾，只需使用这个。

```c
chgat(-1, A_REVERSE, 0, NULL);
```

当更改屏幕上已经存在的字符的属性时，此函数非常有用。移动到要更改的字符并更改属性。

其他函数`wchgat()`， `mvchgat()`， `wchgat()`的行为类似，除了w函数操作在特定的窗口上。mv函数首先移动光标，然后执行给定的工作。实际上，chgat是一个宏，它被一个带有stdscr作为窗口的wchgat()所取代。大多数“无w”函数都是宏。

```c
#include <ncurses.h>

int main(int argc, char *argv[])
{   initscr();          /* Start curses mode        */
    start_color();          /* Start color functionality    */

    init_pair(1, COLOR_CYAN, COLOR_BLACK);
    printw("A Big string which i didn't care to type fully ");
    mvchgat(0, 0, -1, A_BLINK, 1, NULL);    
    /* 
     * First two parameters specify the position at which to start 
     * Third parameter number of characters to update. -1 means till 
     * end of line
     * Forth parameter is the normal attribute you wanted to give 
     * to the charcter
     * Fifth is the color index. It is the index given during init_pair()
     * use 0 if you didn't want color
     * Sixth one is always NULL 
     */
    refresh();
    getch();
    endwin();           /* End curses mode        */
    return 0;
}
```

## Windows

窗口是ncurses中最重要的概念。您已经看到了上面的标准窗口`stdscr`，其中**所有函数都隐式地操作该窗口**。

现在，为了使设计成为最简单的GUI，您需要求助于窗口。使用窗口的主要原因可能是为了单独操作屏幕的各个部分，从而提高效率，只更新需要更改的窗口，从而获得更好的设计。

我想说最后一个原因是选择窗户最重要的。在程序中，您应该始终努力实现更好且易于管理的设计。如果您正在编写大型、复杂的gui，那么在您开始做任何事情之前，这是至关重要的。

### 基础

窗口可以通过调用`newwin()`函数来创建。它不会在屏幕上创建任何东西。它为一个结构分配内存来操作窗口，并更新结构与窗口的数据，如它的大小，开始，开始等。

因此，在curses中，窗口只是一个虚构窗口的抽象，可以独立于屏幕的其他部分进行操作。函数`newwin()`返回一个指向`WINDOW`结构的指针，这个指针可以传递给窗口相关的函数，比如`wprintw()`等。最后，可以使用`delwin()`销毁窗口。它将释放与窗口结构相关联的内存。

### 创建一个窗口

如果一个窗口被创造出来，而我们却看不到它，这是多么有趣。有趣的部分从显示窗口开始。函数`box()`可用于在窗口周围绘制边框。让我们在本例中更详细地研究这些函数。

```c

#include <ncurses.h>

WINDOW *create_newwin(int height, int width, int starty, int startx);
void destroy_win(WINDOW *local_win);

int main(int argc, char *argv[])
{
    WINDOW *my_win;
    int startx, starty, width, height;
    int ch;

    initscr();          /* Start curses mode        */
    cbreak();           /* Line buffering disabled, Pass on everty thing to me */
    keypad(stdscr, TRUE);       /* I need that nifty F1 */

    height = 3;
    width = 10;
    starty = (LINES - height) / 2;  /* Calculating for a center placement */
    startx = (COLS - width) / 2;    /* of the window        */
    printw("Press F1 to exit");
    refresh();
    my_win = create_newwin(height, width, starty, startx);

    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case KEY_LEFT:
                destroy_win(my_win);
                my_win = create_newwin(height, width, starty, --startx);
                break;
            case KEY_RIGHT:
                destroy_win(my_win);
                my_win = create_newwin(height, width, starty, ++startx);
                break;
            case KEY_UP:
                destroy_win(my_win);
                my_win = create_newwin(height, width, --starty, startx);
                break;
            case KEY_DOWN:
                destroy_win(my_win);
                my_win = create_newwin(height, width, ++starty, startx);
                break;  
        }
    }

    endwin();           /* End curses mode        */
    return 0;
}

WINDOW *create_newwin(int height, int width, int starty, int startx)
{   
    WINDOW *local_win;

    local_win = newwin(height, width, starty, startx);
    box(local_win, 0 , 0);      /* 0, 0 gives default characters 
                                 * for the vertical and horizontal
                                 * lines            */
    wrefresh(local_win);        /* Show that box        */

    return local_win;
}

void destroy_win(WINDOW *local_win)
{   
    /* box(local_win, ' ', ' '); : This won't produce the desired
     * result of erasing the window. It will leave it's four corners 
     * and so an ugly remnant of window. 
     */
    wborder(local_win, ' ', ' ', ' ',' ',' ',' ',' ',' ');
    /* The parameters taken are 
     * 1. win: the window on which to operate
     * 2. ls: character to be used for the left side of the window 
     * 3. rs: character to be used for the right side of the window 
     * 4. ts: character to be used for the top side of the window 
     * 5. bs: character to be used for the bottom side of the window 
     * 6. tl: character to be used for the top left corner of the window 
     * 7. tr: character to be used for the top right corner of the window 
     * 8. bl: character to be used for the bottom left corner of the window 
     * 9. br: character to be used for the bottom right corner of the window
     */
    wrefresh(local_win);
    delwin(local_win);
}
```

### 上述代码说明

我知道这是个很好的例子。但我必须在这里解释一些重要的事情:-)。这个程序创建一个矩形窗口，可以用左、右、上、下方向键移动。只要用户按下一个按键，它就会反复创建和破坏窗口。

`create_newwin()`函数的作用是:使用`newwin()`创建一个窗口，并在窗口周围用方框显示边框。函数`destroy_win()`首先用' '字符绘制边框，然后调用`delwin()`来释放与之相关的内存，从而从屏幕上删除窗口。根据用户所按的键，`starty`或`startx`将被更改，并创建一个新窗口。

在`destroy_win`中，如你所见，我使用了`wborder`而不是`box`。原因写在评论里(你错过了。我知道。阅读代码:-))。`wborder`用4个角点和4条线在窗口周围画一个边框。清楚地说，如果你调用`wborder`，如下所示:
```c
wborder(win, '|', '|', '-', '-', '+', '+', '+', '+');
```

显示如下：
```
    +------------+
    |            |
    |            |
    |            |
    |            |
    |            |
    |            |
    +------------+
```

### 例子中其它相关内容

您还可以在上面的示例中看到，我使用了变量`COLS`, `LINES`，它们在`initscr()`之后初始化为屏幕大小。它们可以用于查找屏幕尺寸和查找屏幕的中心坐标。函数`getch()`像往常一样从键盘获取键，并根据键执行相应的工作。这种类型的开关情况在任何基于GUI的程序中都很常见。

### 其它boder函数

上面的程序是非常低效的，因为每按一个键，一个窗口被破坏，另一个窗口被创建。因此，让我们编写一个更有效的程序，使用其他与边界相关的函数。

下面的程序使用`mvhline()`和`mvvline()`来实现类似的效果。这两个函数很简单。它们在指定位置创建指定长度的水平线或垂直线。

```c
#include <ncurses.h>

typedef struct _win_border_struct 
{
    chtype  ls, rs, ts, bs, 
            tl, tr, bl, br;
} WIN_BORDER;

typedef struct _WIN_struct 
{
    int startx, starty;
    int height, width;
    WIN_BORDER border;
} WIN;

void init_win_params(WIN *p_win);
void print_win_params(WIN *p_win);
void create_box(WIN *win, bool flag);

int main(int argc, char *argv[])
{
    WIN win;
    int ch;

    initscr();              /* Start curses mode        */
    start_color();          /* Start the color functionality */
    cbreak();               /* Line buffering disabled, Pass on everty thing to me       */
    keypad(stdscr, TRUE);   /* I need that nifty F1     */
    noecho();
    init_pair(1, COLOR_CYAN, COLOR_BLACK);

    /* Initialize the window parameters */
    init_win_params(&win);
    print_win_params(&win);

    attron(COLOR_PAIR(1));
    printw("Press F1 to exit");
    refresh();
    attroff(COLOR_PAIR(1));

    create_box(&win, TRUE);
    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case KEY_LEFT:
            create_box(&win, FALSE);
            --win.startx;
            create_box(&win, TRUE);
            break;
            case KEY_RIGHT:
            create_box(&win, FALSE);
            ++win.startx;
            create_box(&win, TRUE);
            break;
            case KEY_UP:
            create_box(&win, FALSE);
            --win.starty;
            create_box(&win, TRUE);
            break;
            case KEY_DOWN:
            create_box(&win, FALSE);
            ++win.starty;
            create_box(&win, TRUE);
            break;  
        }
    }
    endwin();           /* End curses mode        */
    return 0;
}

void init_win_params(WIN *p_win)
{
    p_win->height = 3;
    p_win->width = 10;
    p_win->starty = (LINES - p_win->height)/2;  
    p_win->startx = (COLS - p_win->width)/2;

    p_win->border.ls = '|';
    p_win->border.rs = '|';
    p_win->border.ts = '-';
    p_win->border.bs = '-';
    p_win->border.tl = '+';
    p_win->border.tr = '+';
    p_win->border.bl = '+';
    p_win->border.br = '+';

}

void print_win_params(WIN *p_win)
{
#ifdef _DEBUG
    mvprintw(25, 0, "%d %d %d %d", p_win->startx, p_win->starty, 
            p_win->width, p_win->height);
    refresh();
#endif
}

void create_box(WIN *p_win, bool flag)
{   int i, j;
    int x, y, w, h;

    x = p_win->startx;
    y = p_win->starty;
    w = p_win->width;
    h = p_win->height;

    if (flag == TRUE) {
        mvaddch(y, x, p_win->border.tl);
        mvaddch(y, x + w, p_win->border.tr);
        mvaddch(y + h, x, p_win->border.bl);
        mvaddch(y + h, x + w, p_win->border.br);
        mvhline(y, x + 1, p_win->border.ts, w - 1);
        mvhline(y + h, x + 1, p_win->border.bs, w - 1);
        mvvline(y + 1, x, p_win->border.ls, h - 1);
        mvvline(y + 1, x + w, p_win->border.rs, h - 1);

    }
    else {
        for(j = y; j <= y + h; ++j) {
            for(i = x; i <= x + w; ++i) {
                mvaddch(j, i, ' ');
            }
        }
    }

    refresh();
}
```

## 颜色

如您所见，要开始使用颜色，首先应该调用函数`start_color()`。在此之后，您可以使用各种功能来使用终端的颜色功能。要确定终端是否具有颜色功能，可以使用`has_colors()`函数，如果终端不支持颜色，则返回`FALSE`。

Curses在调用`start_color()`时初始化终端支持的所有颜色。这些可以通过定义常量访问，如`COLOR_BLACK`等。现在要真正开始使用颜色，你必须定义颜色对。颜色总是成对使用。这意味着您必须使用`init_pair()`函数为您给出的pair编号定义前台和背景。之后，可以使用`COLOR_PAIR()`函数将对号作为普通属性使用。这可能一开始看起来很麻烦。但是这种优雅的解决方案使我们能够非常容易地管理颜色对。要理解它，你必须研究一下“dialog”的源代码，这是一个用于从shell脚本显示对话框的实用程序。开发人员已经为他们可能需要的所有颜色定义了前景和背景组合，并在开始时进行了初始化。这使得通过访问我们已经定义为常量的一对来设置属性变得非常容易。

下面的颜色在curse.h中定义。您可以使用这些作为各种颜色函数的参数。

```c
COLOR_BLACK   0
COLOR_RED     1
COLOR_GREEN   2
COLOR_YELLOW  3
COLOR_BLUE    4
COLOR_MAGENTA 5
COLOR_CYAN    6
COLOR_WHITE   7
```

### 修改颜色定义

函数`init_color()`可以用来改变最初由`curses`定义的颜色的`rgb`值。假设你想把红色的强度调淡一点。然后你可以用这个函数
```c
init_color(COLOR_RED, 700, 0, 0);

/* param 1     : color name
 * param 2, 3, 4 : rgb content min = 0, max = 1000 */
```

如果您的终端不能更改颜色定义，则该函数返回ERR。函数`can_change_color()`可用于判断终端是否具有更改颜色内容的能力。`rgb`内容从`0`扩展到`1000`。最初定义红色的内容为`1000(r)`， `0(g)`， `0(b)`。

### 颜色上下文

函数`color_content()`和`pair_content()`可用于查找颜色内容和前景色、背景组合。

## 与键盘的接口

没有强大的用户界面，GUI是不完整的，为了与用户交互，curses程序应该对用户的按键或鼠标操作敏感。让我们先处理按键事件。

正如您在上面几乎所有示例中所看到的，从用户那里获得键输入非常容易。获取按键的一个简单方法是使用`getch()`函数。当您感兴趣的是单个按键而不是整行文本(通常以回车符结束)时，应该启用`cbreak`模式来读取键。键盘应该启用功能键，方向键等。有关详细信息，请参阅初始化部分。

`getch()`返回一个与所按键对应的整数。如果是普通字符，则整数值等效于该字符。否则，它返回一个可以与`curses.h`中定义的常量匹配的数字。例如，如果用户按`F1`，返回的整数是`265`。可以使用curses.h中定义的宏`KEY_F()`进行检查。这使得读取键易于携带和管理。

```c
int ch;

ch = getch();
```

`getch()`将等待用户按下一个键(除非您指定了超时)，当用户按下一个键时，将返回相应的整数。然后，您可以检查返回值与curses.h中定义的常量，以匹配所需的键。

```c
if(ch == KEY_LEFT)
    printw("Left arrow is pressed\n");
```

```c
#include <stdio.h>
#include <ncurses.h>

#define WIDTH 30
#define HEIGHT 10 

int startx = 0;
int starty = 0;

char *choices[] = { 
    "Choice 1",
    "Choice 2",
    "Choice 3",
    "Choice 4",
    "Exit",
};

int n_choices = sizeof(choices) / sizeof(char *);
void print_menu(WINDOW *menu_win, int highlight);

int main()
{
    WINDOW *menu_win;
    int highlight = 1;
    int choice = 0;
    int c;

    initscr();
    clear();
    noecho();
    cbreak();   /* Line buffering disabled. pass on everything */
    startx = (80 - WIDTH) / 2;
    starty = (24 - HEIGHT) / 2;

    menu_win = newwin(HEIGHT, WIDTH, starty, startx);
    keypad(menu_win, TRUE);
    mvprintw(0, 0, "Use arrow keys to go up and down, Press enter to select a choice");
    refresh();
    print_menu(menu_win, highlight);
    while(1) {
        c = wgetch(menu_win);
        switch(c) {
            case KEY_UP:
            if(highlight == 1)
                highlight = n_choices;
            else
                --highlight;
            break;
            case KEY_DOWN:
            if(highlight == n_choices)
                highlight = 1;
            else 
                ++highlight;
            break;
            case 10:
            choice = highlight;
            break;
            default:
            mvprintw(24, 0, "Charcter pressed is = %3d Hopefully it can be printed as '%c'", c, c);
            refresh();
            break;
        }
        print_menu(menu_win, highlight);
        if(choice != 0) /* User did a choice come out of the infinite loop */
            break;
    }   
    mvprintw(23, 0, "You chose choice %d with choice string %s\n", choice, choices[choice - 1]);
    clrtoeol();
    refresh();
    endwin();
    return 0;
}


void print_menu(WINDOW *menu_win, int highlight)
{
    int x, y, i;    

    x = 2;
    y = 2;
    box(menu_win, 0, 0);
    for(i = 0; i < n_choices; ++i) {
        if(highlight == i + 1) /* High light the present choice */
        {   wattron(menu_win, A_REVERSE); 
            mvwprintw(menu_win, y, x, "%s", choices[i]);
            wattroff(menu_win, A_REVERSE);
        }
        else
            mvwprintw(menu_win, y, x, "%s", choices[i]);
        ++y;
    }
    wrefresh(menu_win);
}

```

## 鼠标

现在您已经看到了如何获取键，让我们从鼠标执行相同的操作。通常每个UI都允许用户使用键盘和鼠标进行交互。

### 开始

在你做任何其他事情之前，你想要接收的事件必须通过`mousemask()`启用。

```c
mousemask(mmask_t newmask,   /* The events you want to listen to */
        mmask_t *oldmask)    /* The old events mask                */
```

上述函数的第一个参数是您想要侦听的事件的位掩码。默认情况下，所有事件都是关闭的。位掩码`ALL_MOUSE_EVENTS`可用于获取所有事件。

以下是所有的事件掩码:
```c
    Name            Description
   ---------------------------------------------------------------------
   BUTTON1_PRESSED          mouse button 1 down
   BUTTON1_RELEASED         mouse button 1 up
   BUTTON1_CLICKED          mouse button 1 clicked
   BUTTON1_DOUBLE_CLICKED   mouse button 1 double clicked
   BUTTON1_TRIPLE_CLICKED   mouse button 1 triple clicked
   BUTTON2_PRESSED          mouse button 2 down
   BUTTON2_RELEASED         mouse button 2 up
   BUTTON2_CLICKED          mouse button 2 clicked
   BUTTON2_DOUBLE_CLICKED   mouse button 2 double clicked
   BUTTON2_TRIPLE_CLICKED   mouse button 2 triple clicked
   BUTTON3_PRESSED          mouse button 3 down
   BUTTON3_RELEASED         mouse button 3 up
   BUTTON3_CLICKED          mouse button 3 clicked
   BUTTON3_DOUBLE_CLICKED   mouse button 3 double clicked
   BUTTON3_TRIPLE_CLICKED   mouse button 3 triple clicked
   BUTTON4_PRESSED          mouse button 4 down
   BUTTON4_RELEASED         mouse button 4 up
   BUTTON4_CLICKED          mouse button 4 clicked
   BUTTON4_DOUBLE_CLICKED   mouse button 4 double clicked
   BUTTON4_TRIPLE_CLICKED   mouse button 4 triple clicked
   BUTTON_SHIFT             shift was down during button state change
   BUTTON_CTRL              control was down during button state change
   BUTTON_ALT               alt was down during button state change
   ALL_MOUSE_EVENTS         report all button state changes
   REPORT_MOUSE_POSITION    report mouse movement
```

### 获取事件

一旦启用了一类鼠标事件，`getch()`类函数就会在每次发生某个鼠标事件时返回`KEY_MOUSE`。然后可以使用`getmouse()`检索鼠标事件。

```c
MEVENT event;

ch = getch();
if(ch == KEY_MOUSE)
    if(getmouse(&event) == OK)
        /* Do some thing with the event */
        .
        .

```

`getmouse()`将事件返回给它的指针。它是一个包含

```c
typedef struct
{
    short id;         /* ID to distinguish multiple devices */
    int x, y, z;      /* event coordinates */
    mmask_t bstate;   /* button state bits */
}    
```

bstate是我们感兴趣的主要变量。它告诉鼠标的按钮状态。

然后，使用如下所示的代码片段，我们可以了解发生了什么。

```c
if(event.bstate & BUTTON1_PRESSED)
    printw("Left Button Pressed");
```

### 例子

这就是鼠标界面。让我们创建相同的菜单并启用鼠标交互。为了简化操作，删除了键处理。
```c
#include <ncurses.h>

#define WIDTH 30
#define HEIGHT 10 

int startx = 0;
int starty = 0;

char *choices[] = {
    "Choice 1",
    "Choice 2",
    "Choice 3",
    "Choice 4",
    "Exit",
};

int n_choices = sizeof(choices) / sizeof(char *);

void print_menu(WINDOW *menu_win, int highlight);
void report_choice(int mouse_x, int mouse_y, int *p_choice);

int main()
{
    int c, choice = 0;
    WINDOW *menu_win;
    MEVENT event;

    /* Initialize curses */
    initscr();
    clear();
    noecho();
    cbreak();   //Line buffering disabled. pass on everything

    /* Try to put the window in the middle of screen */
    startx = (80 - WIDTH) / 2;
    starty = (24 - HEIGHT) / 2;

    attron(A_REVERSE);
    mvprintw(23, 1, "Click on Exit to quit (Works best in a virtual console)");
    refresh();
    attroff(A_REVERSE);

    /* Print the menu for the first time */
    menu_win = newwin(HEIGHT, WIDTH, starty, startx);
    print_menu(menu_win, 1);

    /* Get all the mouse events */
    mousemask(ALL_MOUSE_EVENTS, NULL);

    while(1) {
        c = wgetch(menu_win);
        switch(c) {
            case KEY_MOUSE:
            if(getmouse(&event) == OK) {
                /* When the user clicks left mouse button */
                if(event.bstate & BUTTON1_PRESSED) {
                    report_choice(event.x + 1, event.y + 1, &choice);
                    if(choice == -1) //Exit chosen
                        goto end;
                    mvprintw(22, 1, "Choice made is : %d String Chosen is \"%10s\"", choice, choices[choice - 1]);
                    refresh(); 
                }
            }
            print_menu(menu_win, choice);
            break;
        }
    }

end:
    endwin();

    return 0;
}

void print_menu(WINDOW *menu_win, int highlight)
{
    int x, y, i;    

    x = 2;
    y = 2;
    box(menu_win, 0, 0);
    for(i = 0; i < n_choices; ++i) {
        if(highlight == i + 1) {
            wattron(menu_win, A_REVERSE); 
            mvwprintw(menu_win, y, x, "%s", choices[i]);
            wattroff(menu_win, A_REVERSE);
        }
        else
            mvwprintw(menu_win, y, x, "%s", choices[i]);
        ++y;
    }
    wrefresh(menu_win);
}

/* Report the choice according to mouse position */
void report_choice(int mouse_x, int mouse_y, int *p_choice)
{   int i,j, choice;

    i = startx + 2;
    j = starty + 3;

    for(choice = 0; choice < n_choices; ++choice)
        if(mouse_y == j + choice && mouse_x >= i && mouse_x <= i + strlen(choices[choice])) {
            if(choice == n_choices - 1)
                *p_choice = -1;     
            else
                *p_choice = choice + 1; 
            break;
        }
}
```
函数`mouse_trafo()`和`wmouse_trafo()`可用于将鼠标坐标转换为屏幕相对坐标。详情请参见`curs_mouse(3X)`手册页。

mouseinterval函数设置按下和释放事件之间的最大时间(以千分之秒为单位)，以便将它们识别为单击。这个函数返回之前的间隔值。默认值是1 / 5秒。

## 屏幕

在本节中，我们将研究一些函数，这些函数使我们能够有效地管理屏幕并编写一些奇特的程序。这在编写游戏时尤其重要。

### getyx()

函数`getyx()`可用于查找当前游标坐标。它将在参数中填充x和y坐标的值。由于`getyx()`是一个宏，您不必传递变量的地址。它可以被称为
```
getyx(win, y, x);
/* win: window pointer
 *   y, x: y, x co-ordinates will be put into this variables 
 */
```

函数`getparyx()`获取子窗口相对于主窗口的起始坐标。这对于更新子窗口有时很有用。当设计一些花哨的东西，如编写多个菜单时，很难存储菜单位置，它们的第一个选项坐标等。这个问题的一个简单解决方案是在子窗口中创建菜单，然后使用`getparyx()`找到菜单的起始坐标。

函数`getbegyx()`和`getmaxyx()`存储当前窗口的起始坐标和最大坐标。这些函数在有效地管理窗口和子窗口方面与上面的方法相同。

### screen 复制

在编写游戏时，有时需要存储屏幕状态并将其恢复到相同的状态。函数`scr_dump()`可用于将屏幕内容转储到作为参数给定的文件中。之后可以通过`scr_restore`函数恢复。这两个简单的功能可以有效地用于保持快速移动的游戏与不断变化的场景。

### window 复制

为了存储和恢复窗口，可以使用`putwin()`和`getwin()`函数。`putwin()`将当前窗口状态放入一个文件，稍后可以通过`getwin()`恢复该文件。

函数`copywin()`可用于将一个窗口完全复制到另一个窗口。它将源窗口和目标窗口作为参数，并根据指定的矩形将矩形区域从源窗口复制到目标窗口。它的最后一个参数指定是将内容覆盖还是覆盖到目标窗口。如果这个论点成立，那么复制是非破坏性的。


## 其它功能点

现在您已经了解了足够多的功能，可以编写一个好的curses程序。有一些杂函数在各种情况下都很有用。让我们来看看其中的一些。

### curs\_set()

此函数可用于使游标不可见。这个函数的参数应该是:
```
0 : invisible      or
1 : normal    or
2 : very visible.
```

### 暂时离开curses模式
有时您可能想暂时回到正常模式(正常线路缓冲模式)。在这种情况下，首先需要通过调用`def_prog_mode()`来保存tty模式，然后调用`endwin()`来结束`curses`模式。这将使您处于原来的tty模式。要在完成之后返回curses，请调用`reset_prog_mode()`。该函数将tty返回到`def_prog_mode()`存储的状态。然后执行`refresh()`，您将回到curses模式。下面的例子显示了要做的事情的顺序。

```c

#include <ncurses.h>

int main()
{   
    initscr();          /* Start curses mode          */
    printw("Hello World !!!\n");    /* Print Hello World          */
    refresh();          /* Print it on to the real screen */
    def_prog_mode();    /* Save the tty modes         */
    endwin();           /* End curses mode temporarily    */

    system("/bin/sh");  /* Do whatever you like in cooked mode */
    reset_prog_mode();  /* Return to the previous tty mode*/

    /* stored by def_prog_mode()      */
    refresh();          /* Do refresh() to restore the    */

    /* Screen contents */
    printw("Another String\n"); /* Back to curses use the full    */
    refresh();          /* capabilities of curses     */
    endwin();           /* End curses mode        */

    return 0;
}
```

### ACS_ 变量


如果您曾经在DOS中编程，您就知道扩展字符集中的那些漂亮字符。它们只能在某些终端上打印。像`box()`这样的NCURSES函数使用这些字符。所有这些变量都以`ACS`开头，即替代字符集。您可能已经注意到我在上面的一些程序中使用了这些字符。下面是一个显示所有字符的示例。


## 其它一些库

除了curses库，还有一些文本模式库，它们提供了更多的功能和许多特性。下面几节将解释通常随curses一起分发的三个标准库。

## Panel 库

既然你精通ncurses，你就想干点大事。您创建了许多重叠的窗口，以提供专业的窗口类型的外观。不幸的是，很快就很难管理这些。多次刷新、更新会让你陷入噩梦。当您忘记按正确的顺序刷新窗口时，重叠的窗口会产生斑点。

不要绝望。在panels库中提供了一个优雅的解决方案。用ncurses开发者的话来说

当您的界面设计使得窗口可能深入可见性堆栈或在运行时弹出到顶部时，所产生的记账可能是乏味的，并且难以正确处理。因此有了面板库。

如果你有很多重叠的窗口，那么面板库是要去的方式。它避免了执行一系列`wnoutrefresh()`，`doupdate()`的需要，并减轻了正确执行(自底向上)的负担。库维护有关窗口顺序、重叠和正确更新屏幕的信息。那为什么还要等待呢?让我们仔细看看面板。

### 基础

Panel 对象是一个窗口，它被隐式地视为包括所有其他Panel对象在内的一部分。deck被视为一个堆栈，顶部Panel是完全可见的，其他Panel可能被遮挡，也可能不被遮挡，根据他们的位置。因此，基本思想是创建一个重叠Panel的堆栈，并使用Panel库来正确显示它们。有一个类似于`refresh()`的函数，当调用它时，将以正确的顺序显示Panel。函数提供隐藏或显示Panel，移动Panel，改变其大小等。在所有对这些函数的调用期间，重叠问题由Panel库管理。

Panel程序的一般流程是这样的:

1. 创建附加到 panel 上的窗口(使用`newwin()`)。
2. 用所选的可见性顺序创建Panel。根据所需的可见性将它们堆叠起来。函数`new_panel()`用于创建面板。
3. 调用`update_panels()`以正确的可见性顺序将面板写入虚拟屏幕。执行`doupdate()`将其显示在屏幕上。
4. 使用`show_panel()`，`hide_panel()`，`move_panel()`等操作面板。使用帮助函数，如`panel_hidden()`和`panel_window()`。使用用户指针存储面板的自定义数据。使用函数`set_panel_userptr()`和`panel_userptr()`来设置和获取面板的用户指针。
5. 当你不再使用panel时，使用`del_panel()`删除。

让我们用一些程序把概念弄清楚。下面是一个简单的程序，它创建了3个重叠的面板，并在屏幕上显示它们。

```c
#include <panel.h>

int main()
{
    WINDOW *my_wins[3];
    PANEL  *my_panels[3];
    int lines = 10, cols = 40, y = 2, x = 4, i;

    initscr();
    cbreak();
    noecho();

    /* Create windows for the panels */
    my_wins[0] = newwin(lines, cols, y, x);
    my_wins[1] = newwin(lines, cols, y + 1, x + 5);
    my_wins[2] = newwin(lines, cols, y + 2, x + 10);

    /* 
     * Create borders around the windows so that you can see the effect
     * of panels
     */
    for(i = 0; i < 3; ++i)
        box(my_wins[i], 0, 0);

    /* Attach a panel to each window */     /* Order is bottom up */
    my_panels[0] = new_panel(my_wins[0]);   /* Push 0, order: stdscr-0 */
    my_panels[1] = new_panel(my_wins[1]);   /* Push 1, order: stdscr-0-1 */
    my_panels[2] = new_panel(my_wins[2]);   /* Push 2, order: stdscr-0-1-2 */

    /* Update the stacking order. 2nd panel will be on top */
    update_panels();

    /* Show it on the screen */
    doupdate();
    
    getch();
    endwin();
}
```

### panel 切换

下面给出了一个稍微复杂的例子。这个程序创建了3个窗口，可以通过使用标签循环。看一下代码。

```c

#include <panel.h>

#define NLINES 10
#define NCOLS 40

void init_wins(WINDOW **wins, int n);
void win_show(WINDOW *win, char *label, int label_color);
void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color);

int main()
{
    WINDOW *my_wins[3];
    PANEL  *my_panels[3];
    PANEL  *top;
    int ch;

    /* Initialize curses */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize all the colors */
    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_GREEN, COLOR_BLACK);
    init_pair(3, COLOR_BLUE, COLOR_BLACK);
    init_pair(4, COLOR_CYAN, COLOR_BLACK);

    init_wins(my_wins, 3);

    /* Attach a panel to each window */     /* Order is bottom up */
    my_panels[0] = new_panel(my_wins[0]);   /* Push 0, order: stdscr-0 */
    my_panels[1] = new_panel(my_wins[1]);   /* Push 1, order: stdscr-0-1 */
    my_panels[2] = new_panel(my_wins[2]);   /* Push 2, order: stdscr-0-1-2 */

    /* Set up the user pointers to the next panel */
    set_panel_userptr(my_panels[0], my_panels[1]);
    set_panel_userptr(my_panels[1], my_panels[2]);
    set_panel_userptr(my_panels[2], my_panels[0]);

    /* Update the stacking order. 2nd panel will be on top */
    update_panels();

    /* Show it on the screen */
    attron(COLOR_PAIR(4));
    mvprintw(LINES - 2, 0, "Use tab to browse through the windows (F1 to Exit)");
    attroff(COLOR_PAIR(4));
    doupdate();

    top = my_panels[2];
    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case 9:
            top = (PANEL *)panel_userptr(top);
            top_panel(top);
            break;
        }
        update_panels();
        doupdate();
    }
    endwin();
    return 0;
}

/* Put all the windows */
void init_wins(WINDOW **wins, int n)
{
    int x, y, i;
    char label[80];

    y = 2;
    x = 10;
    for(i = 0; i < n; ++i) {
        wins[i] = newwin(NLINES, NCOLS, y, x);
        sprintf(label, "Window Number %d", i + 1);
        win_show(wins[i], label, i + 1);
        y += 3;
        x += 7;
    }
}

/* Show the window with a border and a label */
void win_show(WINDOW *win, char *label, int label_color)
{   
    int startx, starty, height, width;

    getbegyx(win, starty, startx);
    getmaxyx(win, height, width);

    box(win, 0, 0);
    mvwaddch(win, 2, 0, ACS_LTEE); 
    mvwhline(win, 2, 1, ACS_HLINE, width - 2); 
    mvwaddch(win, 2, width - 1, ACS_RTEE); 

    print_in_middle(win, 1, 0, width, label, COLOR_PAIR(label_color));
}

void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color)
{  
    int length, x, y;
    float temp;

    if(win == NULL)
        win = stdscr;
    getyx(win, y, x);
    if(startx != 0)
        x = startx;
    if(starty != 0)
        y = starty;
    if(width == 0)
        width = 80;

    length = strlen(string);
    temp = (width - length)/ 2;
    x = startx + (int)temp;
    wattron(win, color);
    mvwprintw(win, y, x, "%s", string);
    wattroff(win, color);
    refresh();
}
```

### panel 的移动和大小改变

函数`move_panel()`可用于将面板移动到所需的位置。它不会改变面板在堆栈中的位置。确保在与面板关联的窗口上使用`move_panel()`而不是`mvwin()`。

调整面板的大小有点复杂。没有直接的函数来调整与面板相关联的窗口的大小。调整面板大小的一个解决方案是创建一个具有所需大小的新窗口，使用`replace_panel()`更改与面板相关的窗口。不要忘记删除旧窗口。可以使用函数`panel_window()`找到与面板关联的窗口。

下面的程序在一个简单的程序中展示了这些概念。您可以像往常一样使用`<TAB>`在窗口中循环。要调整或移动活动面板按`'r'`调整`'m'`移动。然后使用方向键调整大小或将其移动到所需的方式，并按`enter`键结束调整大小或移动。本例使用用户数据获取执行操作所需的数据。

```c

#include <panel.h>

typedef struct _PANEL_DATA {
    int x, y, w, h;
    char label[80]; 
    int label_color;
    PANEL *next;
} PANEL_DATA;

#define NLINES 10
#define NCOLS 40

void init_wins(WINDOW **wins, int n);
void win_show(WINDOW *win, char *label, int label_color);
void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color);
void set_user_ptrs(PANEL **panels, int n);

int main()
{
    WINDOW *my_wins[3];
    PANEL  *my_panels[3];
    PANEL_DATA  *top;
    PANEL *stack_top;
    WINDOW *temp_win, *old_win;
    int ch;
    int newx, newy, neww, newh;
    int size = FALSE, move = FALSE;

    /* Initialize curses */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize all the colors */
    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_GREEN, COLOR_BLACK);
    init_pair(3, COLOR_BLUE, COLOR_BLACK);
    init_pair(4, COLOR_CYAN, COLOR_BLACK);

    init_wins(my_wins, 3);

    /* Attach a panel to each window */     /* Order is bottom up */
    my_panels[0] = new_panel(my_wins[0]);   /* Push 0, order: stdscr-0 */
    my_panels[1] = new_panel(my_wins[1]);   /* Push 1, order: stdscr-0-1 */
    my_panels[2] = new_panel(my_wins[2]);   /* Push 2, order: stdscr-0-1-2 */

    set_user_ptrs(my_panels, 3);
    /* Update the stacking order. 2nd panel will be on top */
    update_panels();

    /* Show it on the screen */
    attron(COLOR_PAIR(4));
    mvprintw(LINES - 3, 0, "Use 'm' for moving, 'r' for resizing");
    mvprintw(LINES - 2, 0, "Use tab to browse through the windows (F1 to Exit)");
    attroff(COLOR_PAIR(4));
    doupdate();

    stack_top = my_panels[2];
    top = (PANEL_DATA *)panel_userptr(stack_top);
    newx = top->x;
    newy = top->y;
    neww = top->w;
    newh = top->h;
    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case 9:     /* Tab */
            top = (PANEL_DATA *)panel_userptr(stack_top);
            top_panel(top->next);
            stack_top = top->next;
            top = (PANEL_DATA *)panel_userptr(stack_top);
            newx = top->x;
            newy = top->y;
            neww = top->w;
            newh = top->h;
            break;
            case 'r':   /* Re-Size*/
            size = TRUE;
            attron(COLOR_PAIR(4));
            mvprintw(LINES - 4, 0, "Entered Resizing :Use Arrow Keys to resize and press <ENTER> to end resizing");
            refresh();
            attroff(COLOR_PAIR(4));
            break;
            case 'm':   /* Move */
            attron(COLOR_PAIR(4));
            mvprintw(LINES - 4, 0, "Entered Moving: Use Arrow Keys to Move and press <ENTER> to end moving");
            refresh();
            attroff(COLOR_PAIR(4));
            move = TRUE;
            break;
            case KEY_LEFT:
            if(size == TRUE)
            {   --newx;
                ++neww;
            }
            if(move == TRUE)
                --newx;
            break;
            case KEY_RIGHT:
            if(size == TRUE)
            {   ++newx;
                --neww;
            }
            if(move == TRUE)
                ++newx;
            break;
            case KEY_UP:
            if(size == TRUE)
            {   --newy;
                ++newh;
            }
            if(move == TRUE)
                --newy;
            break;
            case KEY_DOWN:
            if(size == TRUE)
            {   ++newy;
                --newh;
            }
            if(move == TRUE)
                ++newy;
            break;
            case 10:    /* Enter */
            move(LINES - 4, 0);
            clrtoeol();
            refresh();
            if(size == TRUE)
            {   old_win = panel_window(stack_top);
                temp_win = newwin(newh, neww, newy, newx);
                replace_panel(stack_top, temp_win);
                win_show(temp_win, top->label, top->label_color); 
                delwin(old_win);
                size = FALSE;
            }
            if(move == TRUE)
            {   move_panel(stack_top, newy, newx);
                move = FALSE;
            }
            break;

        }
        attron(COLOR_PAIR(4));
        mvprintw(LINES - 3, 0, "Use 'm' for moving, 'r' for resizing");
        mvprintw(LINES - 2, 0, "Use tab to browse through the windows (F1 to Exit)");
        attroff(COLOR_PAIR(4));
        refresh();  
        update_panels();
        doupdate();
    }
    endwin();
    return 0;
}

/* Put all the windows */
void init_wins(WINDOW **wins, int n)
{  
    int x, y, i;
    char label[80];

    y = 2;
    x = 10;
    for(i = 0; i < n; ++i) {
        wins[i] = newwin(NLINES, NCOLS, y, x);
        sprintf(label, "Window Number %d", i + 1);
        win_show(wins[i], label, i + 1);
        y += 3;
        x += 7;
    }
}

/* Set the PANEL_DATA structures for individual panels */
void set_user_ptrs(PANEL **panels, int n)
{   
    PANEL_DATA *ptrs;
    WINDOW *win;
    int x, y, w, h, i;
    char temp[80];

    ptrs = (PANEL_DATA *)calloc(n, sizeof(PANEL_DATA));

    for(i = 0;i < n; ++i) {
        win = panel_window(panels[i]);
        getbegyx(win, y, x);
        getmaxyx(win, h, w);
        ptrs[i].x = x;
        ptrs[i].y = y;
        ptrs[i].w = w;
        ptrs[i].h = h;
        sprintf(temp, "Window Number %d", i + 1);
        strcpy(ptrs[i].label, temp);
        ptrs[i].label_color = i + 1;
        if(i + 1 == n)
            ptrs[i].next = panels[0];
        else
            ptrs[i].next = panels[i + 1];
        set_panel_userptr(panels[i], &ptrs[i]);
    }
}

/* Show the window with a border and a label */
void win_show(WINDOW *win, char *label, int label_color)
{   
    int startx, starty, height, width;

    getbegyx(win, starty, startx);
    getmaxyx(win, height, width);

    box(win, 0, 0);
    mvwaddch(win, 2, 0, ACS_LTEE); 
    mvwhline(win, 2, 1, ACS_HLINE, width - 2); 
    mvwaddch(win, 2, width - 1, ACS_RTEE); 

    print_in_middle(win, 1, 0, width, label, COLOR_PAIR(label_color));
}

void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color)
{  
    int length, x, y;
    float temp;

    if(win == NULL)
        win = stdscr;
    getyx(win, y, x);
    if(startx != 0)
        x = startx;
    if(starty != 0)
        y = starty;
    if(width == 0)
        width = 80;

    length = strlen(string);
    temp = (width - length)/ 2;
    x = startx + (int)temp;
    wattron(win, color);
    mvwprintw(win, y, x, "%s", string);
    wattroff(win, color);
    refresh();
}
```

注意主要的while循环。一旦它发现所按的键的类型，它就会采取适当的操作。如果按下'r'，则启动调整大小模式。在此之后，随着用户按下方向键，新的大小将被更新。当用户按<ENTER>时，当前选择结束，面板根据所解释的概念调整大小。在调整大小模式下，程序不会显示如何调整窗口大小。当它被调整到一个新位置时，将打印虚线边界作为一个练习留给读者。

当用户按下'm'时，移动模式开始。这比调整大小要简单一些。当方向键被按下时，新的位置会被更新，按`<ENTER>`会通过调用函数`move_panel()`来移动面板。

在这个程序中，表示为`PANEL_DATA`的用户数据在查找与面板相关的信息时起着非常重要的作用。正如注释中所写的，`PANEL_DATA`存储面板大小、标签、标签颜色和循环中指向下一个面板的指针。

### 面板的隐藏和显示

可以使用函数`hide_panel()`隐藏面板。该函数只是将其从面板堆栈中移除，从而在执行`update_panels()`和`doupdate()`时将其隐藏在屏幕上。它不会破坏与隐藏面板相关联的`PANEL`结构。它可以通过使用`show_panel()`函数再次显示。

下面的程序显示了面板的隐藏。按“a”或“b”或“c”分别显示或隐藏第一个、第二个和第三个窗口。它使用带有一个小变量hide的用户数据，该变量跟踪窗口是否被隐藏。由于某种原因，函数`panel_hidden()`没有工作，它告诉你一个面板是否被隐藏。Michael Andres在这里还提供了一份错误报告

```c
#include <panel.h>

typedef struct _PANEL_DATA 
{
    int hide;   /* TRUE if panel is hidden */
} PANEL_DATA;

#define NLINES 10
#define NCOLS 40

void init_wins(WINDOW **wins, int n);
void win_show(WINDOW *win, char *label, int label_color);
void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color);

int main()
{ 
    WINDOW *my_wins[3];
    PANEL  *my_panels[3];
    PANEL_DATA panel_datas[3];
    PANEL_DATA *temp;
    int ch;

    /* Initialize curses */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize all the colors */
    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_GREEN, COLOR_BLACK);
    init_pair(3, COLOR_BLUE, COLOR_BLACK);
    init_pair(4, COLOR_CYAN, COLOR_BLACK);

    init_wins(my_wins, 3);

    /* Attach a panel to each window */     /* Order is bottom up */
    my_panels[0] = new_panel(my_wins[0]);   /* Push 0, order: stdscr-0 */
    my_panels[1] = new_panel(my_wins[1]);   /* Push 1, order: stdscr-0-1 */
    my_panels[2] = new_panel(my_wins[2]);   /* Push 2, order: stdscr-0-1-2 */

    /* Initialize panel datas saying that nothing is hidden */
    panel_datas[0].hide = FALSE;
    panel_datas[1].hide = FALSE;
    panel_datas[2].hide = FALSE;

    set_panel_userptr(my_panels[0], &panel_datas[0]);
    set_panel_userptr(my_panels[1], &panel_datas[1]);
    set_panel_userptr(my_panels[2], &panel_datas[2]);

    /* Update the stacking order. 2nd panel will be on top */
    update_panels();

    /* Show it on the screen */
    attron(COLOR_PAIR(4));
    mvprintw(LINES - 3, 0, "Show or Hide a window with 'a'(first window)  'b'(Second Window)  'c'(Third Window)");
    mvprintw(LINES - 2, 0, "F1 to Exit");

    attroff(COLOR_PAIR(4));
    doupdate();

    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case 'a':           
            temp = (PANEL_DATA *)panel_userptr(my_panels[0]);
            if(temp->hide == FALSE) {
                hide_panel(my_panels[0]);
                temp->hide = TRUE;
            }
            else {
                show_panel(my_panels[0]);
                temp->hide = FALSE;
            }
            break;
            case 'b':
            temp = (PANEL_DATA *)panel_userptr(my_panels[1]);
            if(temp->hide == FALSE) {
                hide_panel(my_panels[1]);
                temp->hide = TRUE;
            }
            else {
                show_panel(my_panels[1]);
                temp->hide = FALSE;
            }
            break;
            case 'c':
            temp = (PANEL_DATA *)panel_userptr(my_panels[2]);
            if(temp->hide == FALSE) {
                hide_panel(my_panels[2]);
                temp->hide = TRUE;
            }
            else {
                show_panel(my_panels[2]);
                temp->hide = FALSE;
            }
            break;
        }
        update_panels();
        doupdate();
    }
    endwin();

    return 0;
}

/* Put all the windows */
void init_wins(WINDOW **wins, int n)
{  
    int x, y, i;
    char label[80];

    y = 2;
    x = 10;
    for(i = 0; i < n; ++i) {
        wins[i] = newwin(NLINES, NCOLS, y, x);
        sprintf(label, "Window Number %d", i + 1);
        win_show(wins[i], label, i + 1);
        y += 3;
        x += 7;
    }
}

/* Show the window with a border and a label */
void win_show(WINDOW *win, char *label, int label_color)
{   
    int startx, starty, height, width;

    getbegyx(win, starty, startx);
    getmaxyx(win, height, width);

    box(win, 0, 0);
    mvwaddch(win, 2, 0, ACS_LTEE); 
    mvwhline(win, 2, 1, ACS_HLINE, width - 2); 
    mvwaddch(win, 2, width - 1, ACS_RTEE); 

    print_in_middle(win, 1, 0, width, label, COLOR_PAIR(label_color));
}

void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color)
{  
    int length, x, y;
    float temp;

    if(win == NULL)
        win = stdscr;
    getyx(win, y, x);
    if(startx != 0)
        x = startx;
    if(starty != 0)
        y = starty;
    if(width == 0)
        width = 80;

    length = strlen(string);
    temp = (width - length)/ 2;
    x = startx + (int)temp;
    wattron(win, color);
    mvwprintw(win, y, x, "%s", string);
    wattroff(win, color);
    refresh();
}
```

### `panel_above()` 和 `panel_below()`

函数`panel_above()`和`panel_below()`可用于查找面板上面和下面的面板。如果这些函数的参数为NULL，则它们分别返回一个指向底部面板和顶部面板的指针。

## Menu 

Menu库为基本的ncurses提供了一个很好的扩展，通过它你可以创建Menu。它提供了一组创建Mneu的函数。但他们必须定制，以提供一个更好的外观，与颜色等。让我们进入细节。

Menu是一种屏幕显示，帮助用户选择给定项目集的某个子集。简单地说，菜单是一个项目的集合，可以从中选择一个或多个项目。有些读者可能不知道多项目选择功能。Menu库提供了编写菜单的功能，用户可以从中选择多个项目作为首选选项。这将在后面的部分中处理。现在是学习一些基本知识的时候了。

要创建Menu，首先要创建item，然后将Menu发布到显示器。在此之后，所有用户响应的处理都在一个优雅的`menu_driver()`函数中完成，它是任何菜单程序的主力。

菜单程序的一般控制流程是这样的:

- 初始化诅咒
- 使用`new_item()`创建项目。您可以为项目指定名称和描述。
- 通过指定要附加的项，使用`new_menu()`创建菜单。
- 使用`menu_post()`发布菜单并刷新屏幕。
- 使用循环处理用户请求，并使用`menu_driver`对菜单进行必要的更新。
- 使用`menu_unpost()`取消菜单
- 释放`free_menu()`分配给菜单的内存
- 使用`free_item()`释放分配给项目的内存
- 结束ncurses

让我们看一个程序，它打印一个简单的菜单，并用上下箭头更新当前选择。

```c
#include <curses.h>
#include <menu.h>

#define ARRAY_SIZE(a) (sizeof(a) / sizeof(a[0]))
#define CTRLD   4

char *choices[] = {
    "Choice 1",
    "Choice 2",
    "Choice 3",
    "Choice 4",
    "Exit",
};

int main()
{
    ITEM **my_items;
    int c;              
    MENU *my_menu;
    int n_choices, i;
    ITEM *cur_item;


    initscr();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    n_choices = ARRAY_SIZE(choices);
    my_items = (ITEM **)calloc(n_choices + 1, sizeof(ITEM *));

    for(i = 0; i < n_choices; ++i)
        my_items[i] = new_item(choices[i], choices[i]);
    my_items[n_choices] = (ITEM *)NULL;

    my_menu = new_menu((ITEM **)my_items);
    mvprintw(LINES - 2, 0, "F1 to Exit");
    post_menu(my_menu);
    refresh();

    while((c = getch()) != KEY_F(1)) {
        switch(c) {
            case KEY_DOWN:
            menu_driver(my_menu, REQ_DOWN_ITEM);
            break;
            case KEY_UP:
            menu_driver(my_menu, REQ_UP_ITEM);
            break;
        }
    }   

    free_item(my_items[0]);
    free_item(my_items[1]);
    free_menu(my_menu);
    endwin();
}
```

本程序演示了使用菜单库创建菜单所涉及的基本概念。首先使用`new_item()`创建项，然后使用`new_menu()`函数将它们附加到菜单。在发布菜单并刷新屏幕之后，主处理循环开始。它读取用户输入并采取相应的操作。`menu_driver()`函数是菜单系统的主要工作对象。这个函数的第二个参数告诉我们要对菜单做什么。根据参数，`menu_driver()`执行相应的任务。该值可以是菜单导航请求、ascii字符或与鼠标事件关联的`KEY_MOUSE`特殊键。

`menu_driver`接受以下导航请求。
```c
REQ_LEFT_ITEM         Move left to an item.
REQ_RIGHT_ITEM      Move right to an item.
REQ_UP_ITEM         Move up to an item.
REQ_DOWN_ITEM       Move down to an item.
REQ_SCR_ULINE       Scroll up a line.
REQ_SCR_DLINE          Scroll down a line.
REQ_SCR_DPAGE          Scroll down a page.
REQ_SCR_UPAGE         Scroll up a page.
REQ_FIRST_ITEM     Move to the first item.
REQ_LAST_ITEM         Move to the last item.
REQ_NEXT_ITEM         Move to the next item.
REQ_PREV_ITEM         Move to the previous item. 
REQ_TOGGLE_ITEM     Select/deselect an item.
REQ_CLEAR_PATTERN     Clear the menu pattern buffer.
REQ_BACK_PATTERN      Delete the previous character from the pattern buffer.
REQ_NEXT_MATCH     Move to the next item matching the pattern match.
REQ_PREV_MATCH     Move to the previous item matching the pattern match.
```
不要被众多的选择压垮。我们将慢慢地看到他们一个接一个。本例中感兴趣的选项是`REQ_UP_ITEM`和`REQ_DOWN_ITEM`。当这两个选项传递给`menu_driver`时，菜单驱动程序将当前项分别更新为向上或向下的一个项。

<br/>

正如您在上面的例子中看到的，`menu_driver`在更新菜单中扮演着重要的角色。理解它所需要的各种选项以及它们的作用是非常重要的。如上所述，`menu_driver()`的第二个参数可以是一个导航请求、一个可打印字符或一个`KEY_MOUSE`键。让我们分析一下不同的导航请求。

||说明|
|---|---|
|`REQ_LEFT_ITEM` 和 `REQ_RIGHT_ITEM`|一个菜单可以显示多个列的一个以上的项目。这可以通过使用`menu_format()`函数来实现。当显示多栏菜单时，这些请求会导致菜单驱动程序将当前选择向左或向右移动。|
|`REQ_UP_ITEM` 和 `REQ_DOWN_ITEM`| 在上面的例子中，您已经看到了这两个选项。当给出这些选项时，使`menu_driver`将当前选择向上或向下移动到一个项目。|
|`ERQ_SCR_*`| `REQ_SCR_ULINE`、`REQ_SCR_DLINE`、`REQ_SCR_DPAGE`、`REQ_SCR_UPAGE`四个选项与滚动有关。如果菜单中的所有项都不能显示在菜单子窗口中，则菜单是可滚动的。这些请求可以交给`menu_driver`来分别上下滚动一行或一页。|
|`REQ_FIRST_ITEM`, `REQ_LAST_ITEM`, `REQ_NEXT_ITEM` 和 `REQ_PREV_ITEM`||
|`REQ_TOGGLE_ITEM`|给出此请求时，切换当前选择。此选项只能在多值菜单中使用。所以要使用这个请求，`O_ONEVALUE`选项必须关闭。可以使用`set_menu_opts()`关闭或打开该选项。|
|pattern requests|每个菜单都有一个相关联的模式缓冲区，用于查找与用户输入的`ascii`字符最近的匹配。只要给`menu_driver`一个`ascii`字符，它就会被放入模式缓冲区。它还尝试在项目列表中找到与模式最接近的匹配项，并将当前选择移动到该项目。请求`REQ_CLEAR_PATTERN`清除模式缓冲区。请求`REQ_BACK_PATTERN`删除模式缓冲区中的前一个字符。如果模式匹配多个项，那么匹配的项可以通过`REQ_NEXT_MATCH`和`REQ_PREV_MATCH`循环，它们分别将当前选择移动到下一个和前一个匹配项。|
|mouse requests|对于`KEY_MOUSE`请求，根据鼠标位置采取相应的操作。要采取的行动在手册页中解释为如果第二个参数是`KEY_MOUSE`特殊键，则相关的鼠标事件被转换为上述事件之一预定义的请求。目前只在用户中单击窗口(如菜单内的显示区域或装饰)Tion窗口)被处理。如果你点击上面的显示区域，则生成`REQ_SCR_ULINE`双击一个`REQ_SCR_UPAGE`生成，如果您点击三下会生成一个`REQ_FIRST_ITEM`。如果你点击在菜单显示区域的下面，是`REQ_SCR_DLINE`生成，如果双击，则会生成`REQ_SCR_DPAGE`如果你点击三次，就会生成一个`REQ_LAST_ITEM`。如果你点击菜单显示区域内的一个项目，菜单光标被定位到该项上。|

以上每个请求都将在适当的情况下在以下几行中通过几个示例进行解释。

### Menu Windows

创建的每个菜单都与一个窗口和一个子窗口相关联。菜单窗口显示与菜单相关的任何标题或边框。菜单子窗口显示当前可供选择的菜单项。但是在这个简单的例子中，我们没有指定任何窗口或子窗口。当没有指定窗口时，将stdscr作为主窗口，然后菜单系统计算显示项目所需的子窗口大小。然后项目显示在计算子窗口中。因此，让我们使用这些窗口并显示一个带有边框和标题的菜单。

这个例子创建了一个带有标题、边框和分隔标题和项的花式线的菜单。如您所见，为了将窗口附加到菜单，必须使用函数`set_menu_win()`。然后我们也附加子窗口。这将显示子窗口中的项目。您还可以使用`set_menu_mark()`设置显示在所选项目左侧的标记字符串。

### 带滚动的菜单

如果给一个窗口的子窗口不够大，不能显示所有的项目，那么菜单将是可滚动的。当您位于当前列表中的最后一项时，如果您发送`REQ_DOWN_ITEM`，它将被转换为`REQ_SCR_DLINE`，并且菜单将按一项滚动。您可以手动给`REQ_SCR_`操作来进行滚动。让我们看看怎么做。

这个程序是不言自明的。在这个例子中，选择的数量已经增加到10个，这比我们的子窗口大小(可以容纳6个项目)要大。必须使用`set_menu_format()`函数显式地将此消息传递给菜单系统。在这里，我们指定要为单个页面显示的行数和列数。我们可以在rows变量中指定要显示的任何数量的项，如果它小于子窗口的高度。如果用户按下的键是`PAGE UP`或`PAGE DOWN`，则由于提供给`menu_driver()`的请求(`REQ_SCR_DPAGE`和`REQ_SCR_UPAGE`)，菜单将滚动一页。

```c
#include <curses.h>
#include <menu.h>

#define ARRAY_SIZE(a) (sizeof(a) / sizeof(a[0]))
#define CTRLD   4

char *choices[] = {
    "Choice 1",
    "Choice 2",
    "Choice 3",
    "Choice 4",
    "Choice 5",
    "Choice 6",
    "Choice 7",
    "Choice 8",
    "Choice 9",
    "Choice 10",
    "Exit",
    (char *)NULL,
};
void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color);

int main()
{
    ITEM **my_items;
    int c;              
    MENU *my_menu;
    WINDOW *my_menu_win;
    int n_choices, i;

    /* Initialize curses */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);
    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_CYAN, COLOR_BLACK);

    /* Create items */
    n_choices = ARRAY_SIZE(choices);
    my_items = (ITEM **)calloc(n_choices, sizeof(ITEM *));
    for(i = 0; i < n_choices; ++i)
        my_items[i] = new_item(choices[i], choices[i]);

    /* Crate menu */
    my_menu = new_menu((ITEM **)my_items);

    /* Create the window to be associated with the menu */
    my_menu_win = newwin(10, 40, 4, 4);
    keypad(my_menu_win, TRUE);

    /* Set main window and sub window */
    set_menu_win(my_menu, my_menu_win);
    set_menu_sub(my_menu, derwin(my_menu_win, 6, 38, 3, 1));
    set_menu_format(my_menu, 5, 1);

    /* Set menu mark to the string " * " */
    set_menu_mark(my_menu, " * ");

    /* Print a border around the main window and print a title */
    box(my_menu_win, 0, 0);
    print_in_middle(my_menu_win, 1, 0, 40, "My Menu", COLOR_PAIR(1));
    mvwaddch(my_menu_win, 2, 0, ACS_LTEE);
    mvwhline(my_menu_win, 2, 1, ACS_HLINE, 38);
    mvwaddch(my_menu_win, 2, 39, ACS_RTEE);

    /* Post the menu */
    post_menu(my_menu);
    wrefresh(my_menu_win);

    attron(COLOR_PAIR(2));
    mvprintw(LINES - 2, 0, "Use PageUp and PageDown to scoll down or up a page of items");
    mvprintw(LINES - 1, 0, "Arrow Keys to navigate (F1 to Exit)");
    attroff(COLOR_PAIR(2));
    refresh();

    while((c = wgetch(my_menu_win)) != KEY_F(1))
    {
        switch(c)
        {
            case KEY_DOWN:
            menu_driver(my_menu, REQ_DOWN_ITEM);
            break;
            case KEY_UP:
            menu_driver(my_menu, REQ_UP_ITEM);
            break;
            case KEY_NPAGE:
            menu_driver(my_menu, REQ_SCR_DPAGE);
            break;
            case KEY_PPAGE:
            menu_driver(my_menu, REQ_SCR_UPAGE);
            break;
        }
        wrefresh(my_menu_win);
    }   

    /* Unpost and free all the memory taken up */
    unpost_menu(my_menu);
    free_menu(my_menu);
    for(i = 0; i < n_choices; ++i)
        free_item(my_items[i]);
    endwin();
}

void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color)
{
    int length, x, y;
    float temp;

    if(win == NULL)
        win = stdscr;
    getyx(win, y, x);
    if(startx != 0)
        x = startx;
    if(starty != 0)
        y = starty;
    if(width == 0)
        width = 80;

    length = strlen(string);
    temp = (width - length)/ 2;
    x = startx + (int)temp;
    wattron(win, color);
    mvwprintw(win, y, x, "%s", string);
    wattroff(win, color);
    refresh();
}
```
### 多栏菜单

在上面的例子中，您已经看到了如何使用`set_menu_format()`函数。我没有提到`cols`变量(第三个参数)的作用。如果子窗口足够宽，您可以选择每行显示多个项目。这可以在`cols`变量中指定。为了简单起见，下面的示例不显示项目的描述。

```c
#include <curses.h>
#include <menu.h>

#define ARRAY_SIZE(a) (sizeof(a) / sizeof(a[0]))
#define CTRLD   4

char *choices[] = {
    "Choice 1", "Choice 2", "Choice 3", "Choice 4", "Choice 5",
    "Choice 6", "Choice 7", "Choice 8", "Choice 9", "Choice 10",
    "Choice 11", "Choice 12", "Choice 13", "Choice 14", "Choice 15",
    "Choice 16", "Choice 17", "Choice 18", "Choice 19", "Choice 20",
    "Exit",
    (char *)NULL,
};

int main()
{
    ITEM **my_items;
    int c;              
    MENU *my_menu;
    WINDOW *my_menu_win;
    int n_choices, i;

    /* Initialize curses */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);
    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_CYAN, COLOR_BLACK);

    /* Create items */
    n_choices = ARRAY_SIZE(choices);
    my_items = (ITEM **)calloc(n_choices, sizeof(ITEM *));
    for(i = 0; i < n_choices; ++i)
        my_items[i] = new_item(choices[i], choices[i]);

    /* Crate menu */
    my_menu = new_menu((ITEM **)my_items);

    /* Set menu option not to show the description */
    menu_opts_off(my_menu, O_SHOWDESC);

    /* Create the window to be associated with the menu */
    my_menu_win = newwin(10, 70, 4, 4);
    keypad(my_menu_win, TRUE);

    /* Set main window and sub window */
    set_menu_win(my_menu, my_menu_win);
    set_menu_sub(my_menu, derwin(my_menu_win, 6, 68, 3, 1));
    set_menu_format(my_menu, 5, 3);
    set_menu_mark(my_menu, " * ");

    /* Print a border around the main window and print a title */
    box(my_menu_win, 0, 0);

    attron(COLOR_PAIR(2));
    mvprintw(LINES - 3, 0, "Use PageUp and PageDown to scroll");
    mvprintw(LINES - 2, 0, "Use Arrow Keys to navigate (F1 to Exit)");
    attroff(COLOR_PAIR(2));
    refresh();

    /* Post the menu */
    post_menu(my_menu);
    wrefresh(my_menu_win);

    while((c = wgetch(my_menu_win)) != KEY_F(1))
    {
        switch(c)
        {
            case KEY_DOWN:
            menu_driver(my_menu, REQ_DOWN_ITEM);
            break;
            case KEY_UP:
            menu_driver(my_menu, REQ_UP_ITEM);
            break;
            case KEY_LEFT:
            menu_driver(my_menu, REQ_LEFT_ITEM);
            break;
            case KEY_RIGHT:
            menu_driver(my_menu, REQ_RIGHT_ITEM);
            break;
            case KEY_NPAGE:
            menu_driver(my_menu, REQ_SCR_DPAGE);
            break;
            case KEY_PPAGE:
            menu_driver(my_menu, REQ_SCR_UPAGE);
            break;
        }
        wrefresh(my_menu_win);
    }   

    /* Unpost and free all the memory taken up */
    unpost_menu(my_menu);
    free_menu(my_menu);
    for(i = 0; i < n_choices; ++i)
        free_item(my_items[i]);
    endwin();
}
```

默认情况下，所有选项都是打开的。您可以使用`menu_opts_on()`和`menu_opts_off()`函数打开或关闭特定的属性。还可以使用`set_menu_opts()`直接指定选项。该函数的实参应该是上述某些常量的一个OR值。函数`menu_opts()`可用于查找菜单的当前选项。

### 多选菜单

您可能想知道如果关闭`O_ONEVALUE`选项会发生什么。然后菜单变成多值。这意味着您可以选择多个项目。这将我们带到请求`REQ_TOGGLE_ITEM`。让我们看看它的实际情况。

```c
#include <curses.h>
#include <menu.h>

#define ARRAY_SIZE(a) (sizeof(a) / sizeof(a[0]))
#define CTRLD   4

char *choices[] = {
    "Choice 1",
    "Choice 2",
    "Choice 3",
    "Choice 4",
    "Choice 5",
    "Choice 6",
    "Choice 7",
    "Exit",
};

int main()
{
    ITEM **my_items;
    int c;              
    MENU *my_menu;
    int n_choices, i;
    ITEM *cur_item;

    /* Initialize curses */ 
    initscr();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize items */
    n_choices = ARRAY_SIZE(choices);
    my_items = (ITEM **)calloc(n_choices + 1, sizeof(ITEM *));
    for(i = 0; i < n_choices; ++i)
        my_items[i] = new_item(choices[i], choices[i]);
    my_items[n_choices] = (ITEM *)NULL;

    my_menu = new_menu((ITEM **)my_items);

    /* Make the menu multi valued */
    menu_opts_off(my_menu, O_ONEVALUE);

    mvprintw(LINES - 3, 0, "Use <SPACE> to select or unselect an item.");
    mvprintw(LINES - 2, 0, "<ENTER> to see presently selected items(F1 to Exit)");
    post_menu(my_menu);
    refresh();

    while((c = getch()) != KEY_F(1))
    {
        switch(c)
        {
            case KEY_DOWN:
            menu_driver(my_menu, REQ_DOWN_ITEM);
            break;
            case KEY_UP:
            menu_driver(my_menu, REQ_UP_ITEM);
            break;
            case ' ':
            menu_driver(my_menu, REQ_TOGGLE_ITEM);
            break;
            case 10:    /* Enter */
            {   char temp[200];
                ITEM **items;

                items = menu_items(my_menu);
                temp[0] = '\0';
                for(i = 0; i < item_count(my_menu); ++i)
                    if(item_value(items[i]) == TRUE)
                    {   strcat(temp, item_name(items[i]));
                        strcat(temp, " ");
                    }
                move(20, 0);
                clrtoeol();
                mvprintw(20, 0, temp);
                refresh();
            }
            break;
        }
    }   

    free_item(my_items[0]);
    free_item(my_items[1]);
    free_menu(my_menu);
    endwin();
}
```

哇，很多新功能。让我们一个接一个地拍。首先，`REQ_TOGGLE_ITEM`。在多值菜单中，应该允许用户选择或取消选择多个项目。请求`REQ_TOGGLE_ITEM`切换当前选择。在这种情况下，当空格被按下时，`REQ_TOGGLE_ITEM`请求被发送到`menu_driver`以实现结果。


现在，当用户按下<ENTER>时，我们将显示他当前选择的项目。首先，我们使用`menu_items()`函数找出与菜单关联的项。然后循环遍历项目，以确定项目是否被选中。函数`item_value()`如果选择了一个项目，则返回TRUE。函数`item_count()`返回菜单中项目的数量。项目名称可以通过`item_name()`找到。您还可以使用`item_description()`找到与项目关联的描述。

### 菜单操作

好吧，到这个时候，你一定渴望你的菜单有一些不同，有很多功能。我知道。你想要颜色!!你想要创建类似于那些文本模式dos游戏的漂亮菜单。函数`set_menu_fore()`和`set_menu_back()`可用于更改选定项和未选定项的属性。这些名字具有误导性。他们不改变菜单的前景或背景，这将是无用的。

函数`set_menu_grey()`可用于设置菜单中不可选择项的显示属性。这为我们带来了一个有趣的项目选项，即唯一的`O_SELECTABLE`。我们可以通过函数`item_opts_off()`关闭它，之后该项就不可选了。它就像那些花哨的窗口菜单上的灰色项目。让我们通过这个例子将这些概念付诸实践

```c

#include <menu.h>

#define ARRAY_SIZE(a) (sizeof(a) / sizeof(a[0]))
#define CTRLD   4

char *choices[] = {
    "Choice 1",
    "Choice 2",
    "Choice 3",
    "Choice 4",
    "Choice 5",
    "Choice 6",
    "Choice 7",
    "Exit",
};

int main()
{
    ITEM **my_items;
    int c;              
    MENU *my_menu;
    int n_choices, i;
    ITEM *cur_item;

    /* Initialize curses */ 
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);
    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_GREEN, COLOR_BLACK);
    init_pair(3, COLOR_MAGENTA, COLOR_BLACK);

    /* Initialize items */
    n_choices = ARRAY_SIZE(choices);
    my_items = (ITEM **)calloc(n_choices + 1, sizeof(ITEM *));
    for(i = 0; i < n_choices; ++i)
        my_items[i] = new_item(choices[i], choices[i]);
    my_items[n_choices] = (ITEM *)NULL;
    item_opts_off(my_items[3], O_SELECTABLE);
    item_opts_off(my_items[6], O_SELECTABLE);

    /* Create menu */
    my_menu = new_menu((ITEM **)my_items);

    /* Set fore ground and back ground of the menu */
    set_menu_fore(my_menu, COLOR_PAIR(1) | A_REVERSE);
    set_menu_back(my_menu, COLOR_PAIR(2));
    set_menu_grey(my_menu, COLOR_PAIR(3));

    /* Post the menu */
    mvprintw(LINES - 3, 0, "Press <ENTER> to see the option selected");
    mvprintw(LINES - 2, 0, "Up and Down arrow keys to naviage (F1 to Exit)");
    post_menu(my_menu);
    refresh();

    while((c = getch()) != KEY_F(1))
    {
        switch(c)
        {
            case KEY_DOWN:
            menu_driver(my_menu, REQ_DOWN_ITEM);
            break;
            case KEY_UP:
            menu_driver(my_menu, REQ_UP_ITEM);
            break;
            case 10: /* Enter */
            move(20, 0);
            clrtoeol();
            mvprintw(20, 0, "Item selected is : %s", 
                    item_name(current_item(my_menu)));
            pos_menu_cursor(my_menu);
            break;
        }
    }   
    unpost_menu(my_menu);
    for(i = 0; i < n_choices; ++i)
        free_item(my_items[i]);
    free_menu(my_menu);
    endwin();
}
```

### 有用的用户指针


我们可以将用户指针与菜单中的每一项关联起来。它的工作方式与面板中的用户指针相同。它不受菜单系统影响。你可以在里面存储任何你喜欢的东西。我通常使用它来存储在选择菜单选项时要执行的函数(它被选中，可能是用户按下<ENTER>);

```c
#include <curses.h>
#include <menu.h>

#define ARRAY_SIZE(a) (sizeof(a) / sizeof(a[0]))
#define CTRLD   4

char *choices[] = {
    "Choice 1",
    "Choice 2",
    "Choice 3",
    "Choice 4",
    "Choice 5",
    "Choice 6",
    "Choice 7",
    "Exit",
};
void func(char *name);

int main()
{
    ITEM **my_items;
    int c;              
    MENU *my_menu;
    int n_choices, i;
    ITEM *cur_item;

    /* Initialize curses */ 
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);
    init_pair(1, COLOR_RED, COLOR_BLACK);
    init_pair(2, COLOR_GREEN, COLOR_BLACK);
    init_pair(3, COLOR_MAGENTA, COLOR_BLACK);

    /* Initialize items */
    n_choices = ARRAY_SIZE(choices);
    my_items = (ITEM **)calloc(n_choices + 1, sizeof(ITEM *));
    for(i = 0; i < n_choices; ++i)
    {       my_items[i] = new_item(choices[i], choices[i]);
        /* Set the user pointer */
        set_item_userptr(my_items[i], func);
    }
    my_items[n_choices] = (ITEM *)NULL;

    /* Create menu */
    my_menu = new_menu((ITEM **)my_items);

    /* Post the menu */
    mvprintw(LINES - 3, 0, "Press <ENTER> to see the option selected");
    mvprintw(LINES - 2, 0, "Up and Down arrow keys to naviage (F1 to Exit)");
    post_menu(my_menu);
    refresh();

    while((c = getch()) != KEY_F(1))
    {
        switch(c)
        {
            case KEY_DOWN:
            menu_driver(my_menu, REQ_DOWN_ITEM);
            break;
            case KEY_UP:
            menu_driver(my_menu, REQ_UP_ITEM);
            break;
            case 10: /* Enter */
            {   ITEM *cur;
                void (*p)(char *);

                cur = current_item(my_menu);
                p = item_userptr(cur);
                p((char *)item_name(cur));
                pos_menu_cursor(my_menu);
                break;
            }
            break;
        }
    }   
    unpost_menu(my_menu);
    for(i = 0; i < n_choices; ++i)
        free_item(my_items[i]);
    free_menu(my_menu);
    endwin();
}

void func(char *name)
{   
    move(20, 0);
    clrtoeol();
    mvprintw(20, 0, "Item selected is : %s", name);
}
```
## 表单（Forms）

好。如果你在网页上看到过这些表单，它们从用户那里获取输入并做各种各样的事情，你可能想知道如何在文本模式下创建这样的表单。用简单的语言写出那些漂亮的形式是相当困难的。表单库试图提供一个基本框架来轻松地构建和维护表单。它有很多特性(功能)，管理验证，动态扩展字段等。让我们来看看它的全部流程。

表单是字段的集合;每个字段可以是一个标签(静态文本)，也可以是一个数据输入位置。表单库还提供了将表单划分为多个页面的函数。

表单的创建方式与菜单的创建方式大致相同。首先使用`new_field()`创建与表单相关的字段。您可以为字段设置选项，这样它们就可以按设置显示一些属性，在字段失去焦点之前进行验证等等。然后将字段附加到表单。在此之后，表单就可以显示并准备接收输入了。在与`menu_driver()`类似的行中，使用`form_driver()`操作表单。我们可以向`form_driver`发送请求，将焦点移动到某个字段，将光标移动到字段的末尾等。在用户在字段中输入值并完成验证后，表单可以取消张贴，分配的内存可以释放。

表单程序的一般控制流程如下所示。

1. 初始化curses
2. 使用`new_field()`创建字段。您可以指定字段的高度和宽度，以及它在表单上的位置。
3. 通过指定要附加的字段，使用`new_form()`创建表单。
4. 使用`form_post()`提交表单并刷新屏幕。
5. 使用循环处理用户请求，并使用`form_driver`对form进行必要的更新。
6. 使用`form_unpost()`取消菜单
7. 释放`free_form()`分配给菜单的内存
8. 使用`free_field()`释放分配给项的内存
9. 结束curses

如您所见，使用表单库与处理菜单库非常相似。下面的示例将探讨表单处理的各个方面。让我们从一个简单的例子开始。

```c
#include <form.h>

int main()
{
    FIELD *field[3];
    FORM  *my_form;
    int ch;

    /* Initialize curses */
    initscr();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize the fields */
    field[0] = new_field(1, 10, 4, 18, 0, 0);
    field[1] = new_field(1, 10, 6, 18, 0, 0);
    field[2] = NULL;

    /* Set field options */
    set_field_back(field[0], A_UNDERLINE);  /* Print a line for the option  */
    field_opts_off(field[0], O_AUTOSKIP);   /* Don't go to next field when this */
    /* Field is filled up       */
    set_field_back(field[1], A_UNDERLINE); 
    field_opts_off(field[1], O_AUTOSKIP);

    /* Create the form and post it */
    my_form = new_form(field);
    post_form(my_form);
    refresh();

    mvprintw(4, 10, "Value 1:");
    mvprintw(6, 10, "Value 2:");
    refresh();

    /* Loop through to get user requests */
    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case KEY_DOWN:
            /* Go to next field */
            form_driver(my_form, REQ_NEXT_FIELD);
            /* Go to the end of the present buffer */
            /* Leaves nicely at the last character */
            form_driver(my_form, REQ_END_LINE);
            break;
            case KEY_UP:
            /* Go to previous field */
            form_driver(my_form, REQ_PREV_FIELD);
            form_driver(my_form, REQ_END_LINE);
            break;
            default:
            /* If this is a normal character, it gets */
            /* Printed                */    
            form_driver(my_form, ch);
            break;
        }
    }

    /* Un post form and free the memory */
    unpost_form(my_form);
    free_form(my_form);
    free_field(field[0]);
    free_field(field[1]); 

    endwin();

    return 0;
}
```

上面的例子很简单。它使用`new_field()`创建了两个字段。`new_field()`获取高度、宽度、起始值、startx、屏幕外行数和额外工作缓冲区数。屏幕外行数的第五个参数指定要显示的字段的大小。如果它为零，则始终显示整个字段，否则当用户访问未显示的字段部分时，表单将是可滚动的。表单库为每个字段分配一个缓冲区来存储用户输入的数据。使用`new_field()`的最后一个参数，我们可以指定它来分配一些额外的缓冲区。这些可以用于任何你喜欢的目的。


在创建字段之后，它们的背景属性都被设置为一个下划线，使用`set_field_back()`。AUTOSKIP选项使用`field_opts_off()`关闭。如果打开此选项，一旦活动字段被完全填充，焦点将移动到表单中的下一个字段。

在将字段附加到表单之后，表单就被发布了。在这里，通过向`form_driver`发出相应的请求，在while循环中处理用户输入。对`form_driver()`的所有请求的详细信息将在后面解释。

每个表单字段都与许多属性相关联。他们可以被操纵，以获得所需的效果和乐趣!!那为什么还要等待呢?

我们在创建字段时给出的参数可以使用`field_info()`检索。它返回高度、宽度、起始值、startx、屏幕外行数以及给定参数的附加缓冲区数。它是`new_field()`的逆函数。

```c
int field_info(     FIELD *field,              /* field from which to fetch */
                    int *height, *int width,   /* field size */ 
                    int *top, int *left,       /* upper left corner */
                    int *offscreen,            /* number of offscreen rows */
                    int *nbuf);                /* number of working buffers */
```

可以使用`move_field()`将字段的位置移动到不同的位置。

```c
int move_field(    FIELD *field,              /* field to alter */
                   int top, int left);        /* new upper-left corner */
```

与往常一样，可以使用`field_infor()`查询更改的位置。

可以使用函数`set_field_just()`对字段进行校正。
```c
int set_field_just(FIELD *field,            /* field to alter */
                    int justmode);          /* mode to set */
int field_just(FIELD *field);               /* fetch justify mode of field */
```

这些函数接受和返回的调整模式值为`NO_JUSTIFICATION`、`justicy_right`、`justicy_left`或`justicy_center`。

### 字段显示属性

如您所见，在上面的示例中，可以使用`set_field_fore()`和`setfield_back()`设置字段的`display`属性。这些函数设置字段的前景和背景属性。您还可以指定一个填充字符，它将被填充在字段的未填充部分。pad字符通过调用`set_field_pad()`来设置。pad默认值为空格。函数`field_fore()`，`field_back`, `field_pad()`可用于查询当前前景，背景属性和字段的填充字符。下面的列表给出了函数的用法。

```c
int set_field_fore(FIELD *field,        /* field to alter */
                   chtype attr);        /* attribute to set */ 

chtype field_fore(FIELD *field);        /* field to query */
                                        /* returns foreground attribute */

int set_field_back(FIELD *field,        /* field to alter */
                   chtype attr);        /* attribute to set */ 

chtype field_back(FIELD *field);        /* field to query */
                                        /* returns background attribute */

int set_field_pad(FIELD *field,         /* field to alter */
                  int pad);             /* pad character to set */ 

chtype field_pad(FIELD *field);         /* field to query */  
                                        /* returns present pad character */
```

虽然上面的函数看起来很简单，但在一开始使用`set_field_fore()`使用颜色可能会令人沮丧。让我先解释一下字段的前景和背景属性。前台属性与字符相关联。这意味着字段中的字符将使用您使用`set_field_fore()`设置的属性打印。背景属性是用于填充字段背景的属性，无论是否有字符。那么颜色呢?由于颜色总是成对定义的，那么显示彩色字段的正确方法是什么?下面是一个说明颜色属性的示例。

```c

#include <form.h>

int main()
{
    FIELD *field[3];
    FORM  *my_form;
    int ch;

    /* Initialize curses */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize few color pairs */
    init_pair(1, COLOR_WHITE, COLOR_BLUE);
    init_pair(2, COLOR_WHITE, COLOR_BLUE);

    /* Initialize the fields */
    field[0] = new_field(1, 10, 4, 18, 0, 0);
    field[1] = new_field(1, 10, 6, 18, 0, 0);
    field[2] = NULL;

    /* Set field options */
    set_field_fore(field[0], COLOR_PAIR(1));/* Put the field with blue background */
    set_field_back(field[0], COLOR_PAIR(2));/* and white foreground (characters */
    /* are printed in white     */
    field_opts_off(field[0], O_AUTOSKIP);   /* Don't go to next field when this */
    /* Field is filled up       */
    set_field_back(field[1], A_UNDERLINE); 
    field_opts_off(field[1], O_AUTOSKIP);

    /* Create the form and post it */
    my_form = new_form(field);
    post_form(my_form);
    refresh();

    set_current_field(my_form, field[0]); /* Set focus to the colored field */
    mvprintw(4, 10, "Value 1:");
    mvprintw(6, 10, "Value 2:");
    mvprintw(LINES - 2, 0, "Use UP, DOWN arrow keys to switch between fields");
    refresh();

    /* Loop through to get user requests */
    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case KEY_DOWN:
            /* Go to next field */
            form_driver(my_form, REQ_NEXT_FIELD);
            /* Go to the end of the present buffer */
            /* Leaves nicely at the last character */
            form_driver(my_form, REQ_END_LINE);
            break;
            case KEY_UP:
            /* Go to previous field */
            form_driver(my_form, REQ_PREV_FIELD);
            form_driver(my_form, REQ_END_LINE);
            break;
            default:
            /* If this is a normal character, it gets */
            /* Printed                */    
            form_driver(my_form, ch);
            break;
        }
    }

    /* Un post form and free the memory */
    unpost_form(my_form);
    free_form(my_form);
    free_field(field[0]);
    free_field(field[1]); 

    endwin();

    return 0;
}
```
使用颜色对，试着理解前景和背景属性。在我使用颜色属性的程序中，我通常只使用`set_field_back()`设置背景。Curses不允许定义单独的颜色属性。

### 字段选项

您还可以设置大量的字段选项位来控制表单处理的各个方面。你可以用这些函数来操作它们:

```c
int set_field_opts(FIELD *field,          /* field to alter */
                   int attr);             /* attribute to set */ 

int field_opts_on(FIELD *field,           /* field to alter */
                  int attr);              /* attributes to turn on */ 

int field_opts_off(FIELD *field,          /* field to alter */
                  int attr);              /* attributes to turn off */ 

int field_opts(FIELD *field);             /* field to query */ 
```
函数`set_field_opts()`可用于直接设置字段的属性，或者您可以选择使用`field_opts_on()`和`field_opts_off()`选择性地打开和关闭一些属性。任何时候都可以使用`field_opts()`查询字段的属性。下面是可用选项的列表。默认情况下，所有选项都是打开的。


<style>table th:first-of-type{width:3cm}</style>

|||
|---|---|
|`O_VISIBLE`|控制该字段在屏幕上是否可见。可用于在表单处理期间根据父字段的值隐藏或弹出字段。|
|`O_ACTIVE`|控制字段在表单处理期间是否处于活动状态(即由表单导航键访问)。可用于使带有缓冲区值的标签或派生字段可由表单应用程序而不是用户更改.|
|`O_PUBLIC`|控制在输入字段时是否显示数据。如果在某个字段上关闭该选项，库将接受并编辑该字段中的数据，但不会显示该数据，可见的字段游标也不会移动。您可以关闭`O_PUBLIC`位来定义密码字段。|
|`O_EDIT`|控制是否可以修改字段的数据。当该选项关闭时，除`REQ_PREV_CHOICE`和`req_next_choice`外的所有编辑请求都将失败。这种只读字段可能对帮助消息有用。|
|`O_WRAP`|控制多行字段的换行。通常，当一个(空白分隔的)单词的任何字符到达当前行的末尾时，整个单词被换行到下一行(假设有一行)。关闭此选项时，单词将在换行符上被分割。|
|`O_BLANK`|控制字段消隐。当该选项打开时，在第一个字段位置输入一个字符将擦除整个字段(除了刚刚输入的字符)。|
|`O_AUTOSKIP`|控制在填充此字段时自动跳转到下一个字段。通常，当表单用户试图在字段中输入超出容量的数据时，编辑位置将跳转到下一个字段。当关闭此选项时，用户的光标将挂在字段的末尾。在未达到其大小限制的动态字段中，此选项将被忽略。|
|`O_NULLOK`|控制是否将验证应用于空白字段。通常情况下，它不是;用户可以在退出时不调用通常的验证检查而将字段留空。如果该选项在字段上关闭，退出该选项将调用验证检查。|
|`O_PASSOK`|控制是否在每个出口上进行验证，还是仅在修改字段之后进行验证。通常后者是正确的。如果字段的验证函数在表单处理过程中发生变化，那么设置`O_PASSOK`可能会很有用。|
|`O_STATIC`|控制字段是否固定为其初始尺寸。如果关闭此选项，该字段将变成动态的，并将拉伸以适应输入的数据。|


当字段当前被选中时，不能更改字段的选项。但是，对于已发布的非当前字段，选项可能会被更改。

选项值是位掩码，可以用逻辑或明显的方式组合。您已经看到了关闭`O_AUTOSKIP`选项的用法。下面的示例说明了更多选项的用法。在适当的地方解释了其他选项。

```c
#include <form.h>

#define STARTX 15
#define STARTY 4
#define WIDTH 25

#define N_FIELDS 3

int main()
{
    FIELD *field[N_FIELDS];
    FORM  *my_form;
    int ch, i;

    /* Initialize curses */
    initscr();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize the fields */
    for(i = 0; i < N_FIELDS - 1; ++i)
        field[i] = new_field(1, WIDTH, STARTY + i * 2, STARTX, 0, 0);
    field[N_FIELDS - 1] = NULL;

    /* Set field options */
    set_field_back(field[1], A_UNDERLINE);  /* Print a line for the option  */

    field_opts_off(field[0], O_ACTIVE); /* This field is a static label */
    field_opts_off(field[1], O_PUBLIC); /* This filed is like a password field*/
    field_opts_off(field[1], O_AUTOSKIP); /* To avoid entering the same field */
    /* after last character is entered */

    /* Create the form and post it */
    my_form = new_form(field);
    post_form(my_form);
    refresh();

    set_field_just(field[0], JUSTIFY_CENTER); /* Center Justification */
    set_field_buffer(field[0], 0, "This is a static Field"); 
    /* Initialize the field  */
    mvprintw(STARTY, STARTX - 10, "Field 1:");
    mvprintw(STARTY + 2, STARTX - 10, "Field 2:");
    refresh();

    /* Loop through to get user requests */
    while((ch = getch()) != KEY_F(1)) {
        switch(ch) {
            case KEY_DOWN:
            /* Go to next field */
            form_driver(my_form, REQ_NEXT_FIELD);
            /* Go to the end of the present buffer */
            /* Leaves nicely at the last character */
            form_driver(my_form, REQ_END_LINE);
            break;
            case KEY_UP:
            /* Go to previous field */
            form_driver(my_form, REQ_PREV_FIELD);
            form_driver(my_form, REQ_END_LINE);
            break;
            default:
            /* If this is a normal character, it gets */
            /* Printed                */    
            form_driver(my_form, ch);
            break;
        }
    }

    /* Un post form and free the memory */
    unpost_form(my_form);
    free_form(my_form);
    free_field(field[0]);
    free_field(field[1]); 

    endwin();

    return 0;
}
```

### 状态字段

字段状态指定字段是否已被编辑。它最初被设置为FALSE，当用户输入一些东西并且数据缓冲区被修改时，它变成TRUE。因此，可以查询字段的状态，以确定它是否已被修改。以下功能可以协助这些操作。

```c
int set_field_status(FIELD *field,      /* field to alter */
                   int status);         /* status to set */
int field_status(FIELD *field);         /* fetch status of field */
```

最好在离开字段后才检查字段的状态，因为数据缓冲区可能还没有更新，因为验证仍然到期。为了保证返回正确的状态，可以调用`field_status()`
1. 在字段的退出验证检查例程中
2. 在字段或表单的初始化或终止钩子中，或者
3. 在表单驱动程序处理完`REQ_VALIDATION`请求之后

### 用户指针字段

每个字段结构都包含一个指针，用户可以将其用于各种目的。它不受表单库的影响，用户可以将其用于任何目的。下面的函数设置和获取用户指针。

```c
int set_field_userptr(FIELD *field,   
           char *userptr);      /* the user pointer you wish to associate */
                                /* with the field    */

char *field_userptr(FIELD *field);      /* fetch user pointer of the field */
```

### Variable大小字段

如果您想要一个具有可变宽度的动态变化字段，这就是您想要充分利用的特性。这将允许用户输入比字段原始大小更多的数据，并让字段增长。根据字段方向，它将水平或垂直滚动以合并新数据。

要使字段可动态增长，应关闭`O_STATIC`选项。这可以用

```c
field_opts_off(field_pointer, O_STATIC);
```

但是，让一块地无限生长通常是不可取的。您可以设置一个最大限制的增长领域与

```c
int set_max_field(FIELD *field,    /* Field on which to operate */
                  int max_growth); /* maximum growth allowed for the field */
```

动态增长字段的字段信息可以通过

```c
int dynamic_field_info( FIELD *field,     /* Field on which to operate */
            int   *prows,     /* number of rows will be filled in this */
            int   *pcols,     /* number of columns will be filled in this*/
            int   *pmax)      /* maximum allowable growth will be filled */
                              /* in this */
```

尽管`field_info`工作正常，但建议使用此函数获取动态增长字段的适当属性。

调用标准库例程`new_field`;高度设置为1的新字段将被定义为一行字段。高度大于1的新字段将被定义为多行字段。

关闭`O_STATIC`的一行字段(动态可增长字段)将包含单个固定行，但如果用户输入的数据超过初始字段所能容纳的数据，则列的数量可能会增加。显示的列数将保持固定，额外的数据将水平滚动。

关闭`O_STATIC`的多行字段(动态可增长字段)将包含固定数量的列，但如果用户输入的数据多于初始字段所容纳的数据，则行数可以增加。显示的行数将保持固定，额外的数据将垂直滚动。

上面两段描述了动态增长字段的行为。表单库其他部分的行为方式如下所示:

1. 如果`O_STATIC`选项关闭，并且没有为字段指定最大增长，那么字段选项`O_AUTOSKIP`将被忽略。目前，当用户输入字段的最后一个字符位置时，`O_AUTOSKIP`会自动生成一个`REQ_NEXT_FIELD`表单驱动程序请求。在没有指定最大增长的可增长字段上，没有最后一个字符位置。如果指定了最大增长，那么如果字段已经增长到最大大小，`O_AUTOSKIP`选项将正常工作。
2. 如果选项`O_STATIC`被关闭，字段的校正将被忽略。目前，`set_field_just`可以用来对一行字段的内容进行`justicy_left`, `justicy_right`, `justicy_center`。根据定义，可增长的一行字段将水平增长和滚动，并且可能包含超出合理范围的数据。`field_just`的返回值将保持不变。
3. 如果字段选项`O_STATIC`关闭，并且没有为字段指定最大增长，重载表单驱动程序请求`REQ_NEW_LINE`将以相同的方式操作，而不管`O_NL_OVERLOAD`表单选项。目前，如果表单选项`O_NL_OVERLOAD`是打开的，那么如果从字段的最后一行调用`REQ_NEW_LINE`，则隐式生成一个`REQ_NEXT_FIELD`。如果字段可以无限制地增长，则没有最后一行，因此`REQ_NEW_LINE`永远不会隐式生成`REQ_NEXT_FIELD`。如果指定了最大增长限制，并且打开了`O_NL_OVERLOAD`表单选项，则`REQ_NEW_LINE`仅在字段已增长到最大大小且用户位于最后一行时才隐式生成`REQ_NEXT_FIELD`。
4. 库调用`dup_field`将照常工作;它将复制字段，包括当前缓冲区大小和被复制字段的内容。任何指定的最大增长也将被复制。
5. 标准库调用`link_field`将照常工作;它将复制所有字段属性，并与被链接的字段共享缓冲区。如果`O_STATIC`字段选项随后被一个字段共享缓冲区更改，那么当试图向字段中输入比缓冲区当前持有的数据更多的数据时，系统的反应将取决于当前字段中该选项的设置。
6. 标准库调用`field_info`将照常工作;变量nrow将包含对`new_field`的原始调用的值。用户应该使用上面描述的`dynamic_field_info`来查询缓冲区的当前大小。

只有在解释了表单驱动程序之后，上面的一些观点才有意义。我们将在接下来的几节中讨论这个问题。

### 表格界面

表单窗口的概念与菜单窗口非常相似。每个表单都与一个主窗口和一个子窗口相关联。表单主窗口显示任何相关的标题或边框或任何用户希望的内容。然后子窗口包含所有字段，并根据它们的位置显示它们。这就提供了非常容易地操纵花哨形式显示的灵活性。

因为这与菜单窗口非常相似，所以我只提供了一个例子，不做太多解释。它们的功能相似，工作方式相同。

```c
#include <form.h>

void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color);

int main()
{
    FIELD *field[3];
    FORM  *my_form;
    WINDOW *my_form_win;
    int ch, rows, cols;

    /* Initialize curses */
    initscr();
    start_color();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);

    /* Initialize few color pairs */
    init_pair(1, COLOR_RED, COLOR_BLACK);

    /* Initialize the fields */
    field[0] = new_field(1, 10, 6, 1, 0, 0);
    field[1] = new_field(1, 10, 8, 1, 0, 0);
    field[2] = NULL;

    /* Set field options */
    set_field_back(field[0], A_UNDERLINE);
    field_opts_off(field[0], O_AUTOSKIP); /* Don't go to next field when this */
    /* Field is filled up         */
    set_field_back(field[1], A_UNDERLINE); 
    field_opts_off(field[1], O_AUTOSKIP);

    /* Create the form and post it */
    my_form = new_form(field);

    /* Calculate the area required for the form */
    scale_form(my_form, &rows, &cols);

    /* Create the window to be associated with the form */
    my_form_win = newwin(rows + 4, cols + 4, 4, 4);
    keypad(my_form_win, TRUE);

    /* Set main window and sub window */
    set_form_win(my_form, my_form_win);
    set_form_sub(my_form, derwin(my_form_win, rows, cols, 2, 2));

    /* Print a border around the main window and print a title */
    box(my_form_win, 0, 0);
    print_in_middle(my_form_win, 1, 0, cols + 4, "My Form", COLOR_PAIR(1));

    post_form(my_form);
    wrefresh(my_form_win);

    mvprintw(LINES - 2, 0, "Use UP, DOWN arrow keys to switch between fields");
    refresh();

    /* Loop through to get user requests */
    while((ch = wgetch(my_form_win)) != KEY_F(1)) {
        switch(ch) {
            case KEY_DOWN:
            /* Go to next field */
            form_driver(my_form, REQ_NEXT_FIELD);
            /* Go to the end of the present buffer */
            /* Leaves nicely at the last character */
            form_driver(my_form, REQ_END_LINE);
            break;
            case KEY_UP:
            /* Go to previous field */
            form_driver(my_form, REQ_PREV_FIELD);
            form_driver(my_form, REQ_END_LINE);
            break;
            default:
            /* If this is a normal character, it gets */
            /* Printed                */    
            form_driver(my_form, ch);
            break;
        }
    }

    /* Un post form and free the memory */
    unpost_form(my_form);
    free_form(my_form);
    free_field(field[0]);
    free_field(field[1]); 

    endwin();
    return 0;
}

void print_in_middle(WINDOW *win, int starty, int startx, int width, char *string, chtype color)
{   
    int length, x, y;
    float temp;

    if(win == NULL)
        win = stdscr;
    getyx(win, y, x);
    if(startx != 0)
        x = startx;
    if(starty != 0)
        y = starty;
    if(width == 0)
        width = 80;

    length = strlen(string);
    temp = (width - length)/ 2;
    x = startx + (int)temp;
    wattron(win, color);
    mvwprintw(win, y, x, "%s", string);
    wattroff(win, color);
    refresh();
}
```

### 字段验证

默认情况下，字段将接受用户输入的任何数据。可以将验证附加到字段。然后，当该字段包含与验证类型不匹配的数据时，用户试图离开该字段的任何尝试都将失败。某些验证类型还在字段中每次输入字符时进行字符有效性检查。

可以使用以下函数将验证附加到字段。

```c
int set_field_type(FIELD *field,          /* field to alter */
                   FIELDTYPE *ftype,      /* type to associate */
                   ...);                  /* additional arguments*/
```

设置后，可以使用该字段的验证类型进行查询

```c
FIELDTYPE *field_type(FIELD *field);      /* field to query */
```

表单驱动程序仅在最终用户输入数据时验证字段中的数据。时不发生验证

- 应用程序通过调用`set_field_buffer`来更改字段值。
- 链接的字段值是间接改变的——通过改变它们链接到的字段

下面是预定义的验证类型。您还可以指定自定义验证，尽管这有点棘手和麻烦。

#### TYPE\_ALPHA

此字段类型接受字母数据;没有空格，没有数字，没有特殊字符(在字符输入时检查)。它的设置为:

```c
int set_field_type(FIELD *field,          /* field to alter */
                   TYPE_ALPHA,            /* type to associate */
                   int width);            /* maximum width of field */
```

width参数设置数据的最小宽度。用户必须输入至少宽度的字符数才能离开字段。通常你会把这个设置为字段宽度;如果它大于字段宽度，验证检查将始终失败。最小宽度为零使得字段补全是可选的。

#### TYPE\_ALNUM

此字段类型接受字母数据和数字;没有空格，没有特殊字符(在字符输入时检查)。它的设置为:

```c
int set_field_type(FIELD *field,          /* field to alter */
                   TYPE_ALNUM,            /* type to associate */
                   int width);            /* maximum width of field */
```

width参数设置数据的最小宽度。与`TYPE_ALPHA`一样，通常你需要将其设置为字段宽度;如果它大于字段宽度，验证检查将始终失败。最小宽度为零使得字段补全是可选的。

#### TYPE\_ENUM

此类型允许您将字段的值限制为指定的字符串值集(例如，美国各州的两个字母邮政编码)。它的设置为:

```c
int set_field_type(FIELD *field,          /* field to alter */
                   TYPE_ENUM,             /* type to associate */
                   char **valuelist;      /* list of possible values */
                   int checkcase;         /* case-sensitive? */
                   int checkunique);      /* must specify uniquely? */
```

valuelist参数必须指向以null结束的有效字符串列表。checkcase参数如果为真，则与字符串区分大小写进行比较。

当用户退出TYPE_ENUM字段时，验证过程尝试将缓冲区中的数据填充为有效项。如果输入了完整的选择字符串，它当然是有效的。但是也可以输入一个有效字符串的前缀并为您完成它。

缺省情况下，如果输入的前缀在字符串列表中匹配了一个以上的值，则从第一个匹配的值开始补全。但是checkunique参数如果为真，则要求前缀匹配是唯一的，以便有效。

REQ_NEXT_CHOICE和REQ_PREV_CHOICE输入请求对于这些字段特别有用。

#### TYPE_INTEGER

此字段类型接受一个整数。其设置如下:

```c
int set_field_type(FIELD *field,          /* field to alter */
                   TYPE_INTEGER,          /* type to associate */
                   int padding,           /* # places to zero-pad to */
                   int vmin, int vmax);   /* valid range */
```

有效字符由可选的前导减号和数字组成。在退出时执行范围检查。如果范围最大值小于或等于最小值，则忽略范围。

如果值通过了范围检查，它将填充尽可能多的前导零数字，以满足填充参数。

TYPE_INTEGER值缓冲区可以用C库函数atoi(3)方便地解释。

#### TYPE_NUMERIC

此字段类型接受十进制数。其设置如下:

```c
int set_field_type(FIELD *field,          /* field to alter */
                   TYPE_NUMERIC,          /* type to associate */
                   int padding,           /* # places of precision */
                   int vmin, int vmax);   /* valid range */
```

有效字符由可选的前导减号和数字组成。可能还包括一个小数点。在退出时执行范围检查。如果范围最大值小于或等于最小值，则忽略范围。

如果值通过了范围检查，它将填充尽可能多的尾随零数字，以满足填充参数。

TYPE_NUMERIC值缓冲区可以用C库函数atof(3)方便地解释。

#### TYPE_REHEXP

此字段类型接受匹配正则表达式的数据。其设置如下:

```c
int set_field_type(FIELD *field,          /* field to alter */
                   TYPE_REGEXP,           /* type to associate */
                   char *regexp);         /* expression to match */
```

正则表达式的语法是regcomp(3)。正则表达式匹配检查在退出时执行。

### 表单驱动——表单系统主力军

和菜单系统一样，form_driver()在表单系统中扮演着非常重要的角色。对表单系统的所有类型的请求都应该通过form_driver()过滤。

```c
int form_driver(FORM *form,     /* form on which to operate     */
                int request)    /* form request code         */
```

正如您已经看到的上面的一些示例，您必须在循环中寻找用户输入，然后决定它是字段数据还是表单请求。然后表单请求被传递给form_driver()来完成这项工作。

请求大致可以分为以下几类。下面解释不同的请求及其用法:

### 页面导航请求 

这些请求导致页面级的表单移动，触发新表单屏幕的显示。一个表单可以由多个页面组成。如果您有一个包含许多字段和逻辑部分的大表单，那么您可以将表单划分为页面。函数set_new_page()在指定的字段处设置一个新页面。

```c
int set_new_page(FIELD *field,/* Field at which page break to be set or unset */
         bool new_page_flag); /* should be TRUE to put a break */
```

下面的请求允许您移动到不同的页面

- REQ_NEXT_PAGE 移动到下一个表单页面。
- REQ_PREV_PAGE 移动到上一个表单页面。
- REQ_FIRST_PAGE 移动到第一个表单页面。
- REQ_LAST_PAGE 移动到最后一个表单页面。

这些请求将列表视为循环的;也就是说，从最后一页的REQ_NEXT_PAGE到第一页，从第一页的REQ_PREV_PAGE到最后一页。

### 字段内导航请求

- REQ_NEXT_FIELD 移动到下一个字段。
- REQ_PREV_FIELD 移动到上一个字段。
- REQ_FIRST_FIELD 移动到第一个字段。
- REQ_LAST_FIELD 移动到最后一个字段。
- REQ_SNEXT_FIELD 移动到已排序的下一个字段。
- REQ_SPREV_FIELD 移动到排序前一个字段。
- REQ_SFIRST_FIELD REQ_SFIRST_FIELD移动到排序的第一个字段。
- REQ_SLAST_FIELD移动到最后一个排序字段。
- REQ_LEFT_FIELD 向左移动到字段。
- REQ_RIGHT_FIELD 右移到字段。
- REQ_UP_FIELD上移到field。
- REQ_DOWN_FIELD 向下移动到字段。

这些请求将页面上的字段列表作为循环处理;也就是说，从最后一个字段的REQ_NEXT_FIELD到第一个字段，从第一个字段的REQ_PREV_FIELD到最后一个字段。这些(以及REQ_FIRST_FIELD和REQ_LAST_FIELD请求)字段的顺序就是表单数组中字段指针的顺序(由new_form()或set_form_fields()设置)

也可以遍历字段，就像它们按照屏幕位置顺序排序一样，因此序列从左到右，从上到下。为此，使用第二组4个排序移动请求。

最后，还可以使用上、下、右、左的可视方向在字段之间移动。要实现这一点，请使用第三组请求。但是请注意，表单用于这些请求的位置是其左上角。

例如，假设您有一个多行字段B，两个单行字段a和C与B在同一行，a在B的左边，C在B的右边。只有当a、B和C共享相同的第一行时，a的REQ_MOVE_RIGHT才会转到B;否则它将跳过B到C。


These requests drive movement of the edit cursor within the currently selected field.

    REQ_NEXT_CHAR Move to next character.

    REQ_PREV_CHAR Move to previous character.

    REQ_NEXT_LINE Move to next line.

    REQ_PREV_LINE Move to previous line.

    REQ_NEXT_WORD Move to next word.

    REQ_PREV_WORD Move to previous word.

    REQ_BEG_FIELD Move to beginning of field.

    REQ_END_FIELD Move to end of field.

    REQ_BEG_LINE Move to beginning of line.

    REQ_END_LINE Move to end of line.

    REQ_LEFT_CHAR Move left in field.

    REQ_RIGHT_CHAR Move right in field.

    REQ_UP_CHAR Move up in field.

    REQ_DOWN_CHAR Move down in field. 

Each word is separated from the previous and next characters by whitespace. The commands to move to beginning and end of line or field look for the first or last non-pad character in their ranges.
18.6.4. Scrolling Requests

Fields that are dynamic and have grown and fields explicitly created with offscreen rows are scrollable. One-line fields scroll horizontally; multi-line fields scroll vertically. Most scrolling is triggered by editing and intra-field movement (the library scrolls the field to keep the cursor visible). It is possible to explicitly request scrolling with the following requests:

    REQ_SCR_FLINE Scroll vertically forward a line.

    REQ_SCR_BLINE Scroll vertically backward a line.

    REQ_SCR_FPAGE Scroll vertically forward a page.

    REQ_SCR_BPAGE Scroll vertically backward a page.

    REQ_SCR_FHPAGE Scroll vertically forward half a page.

    REQ_SCR_BHPAGE Scroll vertically backward half a page.

    REQ_SCR_FCHAR Scroll horizontally forward a character.

    REQ_SCR_BCHAR Scroll horizontally backward a character.

    REQ_SCR_HFLINE Scroll horizontally one field width forward.

    REQ_SCR_HBLINE Scroll horizontally one field width backward.

    REQ_SCR_HFHALF Scroll horizontally one half field width forward.

    REQ_SCR_HBHALF Scroll horizontally one half field width backward. 

For scrolling purposes, a page of a field is the height of its visible part.
18.6.5. Editing Requests

When you pass the forms driver an ASCII character, it is treated as a request to add the character to the field's data buffer. Whether this is an insertion or a replacement depends on the field's edit mode (insertion is the default.

The following requests support editing the field and changing the edit mode:

    REQ_INS_MODE Set insertion mode.

    REQ_OVL_MODE Set overlay mode.

    REQ_NEW_LINE New line request (see below for explanation).

    REQ_INS_CHAR Insert space at character location.

    REQ_INS_LINE Insert blank line at character location.

    REQ_DEL_CHAR Delete character at cursor.

    REQ_DEL_PREV Delete previous word at cursor.

    REQ_DEL_LINE Delete line at cursor.

    REQ_DEL_WORD Delete word at cursor.

    REQ_CLR_EOL Clear to end of line.

    REQ_CLR_EOF Clear to end of field.

    REQ_CLR_FIELD Clear entire field. 

REQ_NEW_LINE和REQ_DEL_PREV请求的行为很复杂，部分由一对表单选项控制。当游标位于字段的开头或字段的最后一行时，会触发特殊情况。

首先，我们考虑REQ_NEW_LINE:

REQ_NEW_LINE在插入模式下的正常行为是在编辑游标的位置打断当前行，在游标之后插入当前行的部分，作为跟随当前的新行，并将游标移动到新行的开头(您可以认为这是在字段缓冲区中插入换行符)。

REQ_NEW_LINE在叠加模式下的正常行为是清除当前行从编辑光标到行尾的位置。然后将光标移动到下一行的开头。

但是，字段开头或字段最后一行的REQ_NEW_LINE将执行REQ_NEXT_FIELD。O_NL_OVERLOAD选项关闭，此特殊动作被禁用。

现在，让我们考虑REQ_DEL_PREV:

REQ_DEL_PREV的正常行为是删除前一个字符。如果插入模式是打开的，并且光标位于一行的开头，并且该行上的文本将与前一行相匹配，那么它会将当前行的内容追加到前一行并删除当前行(您可以将此视为从字段缓冲区中删除换行符)。

但是，字段开头的REQ_DEL_PREV被视为REQ_PREV_FIELD。

如果O_BS_OVERLOAD选项被关闭，这个特殊的动作将被禁用，表单驱动程序只返回E_REQUEST_DENIED。

### 按顺序请求

如果你的字段类型是有序的，并且有相关的函数来从给定的值中获取该类型的下一个和上一个值，有一些请求可以将该值获取到字段缓冲区:

- REQ_NEXT_CHOICE 将当前值的后继值放入缓冲区。
- REQ_PREV_CHOICE 将当前值的前任值放在缓冲区中。

在内置字段类型中，只有TYPE_ENUM具有内置的后继函数和前任函数。当您定义自己的字段类型(请参阅自定义验证类型)时，您可以关联我们自己的排序函数。

### 程序命令

表单请求表示为curses值大于KEY_MAX且小于或等于常量MAX_COMMAND的整数。在这个范围内的值将被form_driver()忽略。这可以被应用程序用于任何目的。它可以被视为应用程序特定的操作，并采取相应的操作。

## ncurses 工具和窗口库

现在，您已经看到了ncurses及其姐妹库的功能，您可以卷起袖子，为一个大量操纵屏幕的项目做准备了。但是等等. .在普通的ncurses中编写和维护复杂的GUI小部件，甚至使用附加的库都是相当困难的。有一些现成的工具和小部件库，可以用来代替编写自己的小部件。您可以使用其中的一些，从代码中获取想法，甚至扩展它们。

### CDK(curses Development Kit)

用作者的话来说，CDK代表“curses开发工具包”，它目前包含21个准备使用的小部件，以促进全屏curses程序的快速开发。

该工具包提供了一些有用的小部件，可以直接在程序中使用。它写得很好，文档也很好。examples目录中的示例对于初学者来说是一个很好的开始。CDK可以从http://invisible-island.net/cdk/下载。按照README文件中的说明安装它。

### Widget List

下面是cdk提供的小部件列表及其描述。

|Widget类型|描述|
|---|---|
|`Alphalist`|允许用户从单词列表中选择属性来缩小搜索列表的范围所需单词的几个字符。|
|`Buttonbox`|创建多按钮widget|
|`Calendar`|创建日历窗口|
|`Dialog`| 用消息提示用户，而用户可以从提供的按钮中选择答案。|
|`Entry`|允许用户输入|
|`File Selector`|从Cdk基本小部件构建的文件选择器。这示例显示如何创建更复杂的小部件使用Cdk小部件库。|
|`Graph`|绘制图形|
|`Histogram`||
|`Item List`|创建一个弹出字段，允许用户进行选择在一个小领域的几个选择之一。非常有用的比如星期几或月份的名称。|
|`Label`|在弹出框中显示消息，或者标签可以是被视为屏幕的一部分。|
|`Marquee`|在滚动字幕中显示消息。|
|`Matrix`|创建一个复杂的矩阵与许多选项|
|`Menu`|创建一个下拉菜单界面。|
|`Multiple Line Entry`|多行输入字段。非常有用的对于长字段。(像描述字段)|
|`Radio List`|创建单选按钮列表。|
|`Scale`|创建数字刻度。用于允许用户选择一个数值，并将其限制在值。|
|`Scrolling List`|创建一个滚动列表/菜单列表。|
|`Scrolling Window`|创建滚动日志文件查看器。可以添加信息进入窗口，而它的运行。一个用于显示进度的好小部件某物(类似于控制台窗口)|
|`Selection List`|创建多个选项选择列表。|
|`Slider`|类似于缩放小部件，此小部件提供了一个可视滑条来表示数值。|
|`Template`|创建字符敏感的输入字段的位置。用于预先格式化的字段，如日期和电话号码。|
|`Viewer`|这是一个文件/信息查看器。非常有用的当你需要显示大量信息时。|

Thomas Dickey在最近的版本中修改了一些小部件。

