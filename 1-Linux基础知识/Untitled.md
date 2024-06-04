

## 什么是终端(Terminal)

在 Linux 中，终端（Terminal）是用户与操作系统进行交互的界面。它提供了一个命令行界面（Command Line Interface，简称 CLI），用户可以通过输入命令来操作系统、运行程序和执行各种任务。



## 什么是shell

### 1、shell的定义

在 `unix`系统中（Linux系统派生自该系统中，Linux系统可以成为`unix-like(类Unix系统)`），用来解释和管理命令的程序被称为 shell；

### 2、 shell的种类：

![image-20230825144920865](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/image-20230825144920865.png)

- `Bourne Shell (sh):` 1979年，由 Stephen Bourne 开发，第一个广泛使用的 Unix shell，语法简单，功能有限；
- `C Shell (csh):` 1979年，由 Bill Joy 开发，提供了更多的交互功能和命令历史记录，支持自定义命令别名和变量；
- `korn Sehll (ksh):` 1983年，由David Korn开发，结合了Bourne Shell 和 C Shell 的特点，引入了循环结构、算数运算和命令行编辑功能；
- `Bourne Again Shell (bash):` 1989年，由 Brian Fox 开发，GNU 项目中默认 shell，兼容 Bourne Shell，并提供了更多的功能和扩展性；
- `Z Shell (zsh):` 1990年，由 Paul Falstad 开发，提供了更强大的命令行编辑和自动补全功能，支持更多的配置选项和插件；
- `Fish Shell (fish):` 2005年，由 Axel Liljencrantz 开发，设计目标是易用性和用户友好性，提供了智能补全和语法高亮等功能；
- `PowerShell:` 2006年，由 Microsoft 开发，面向 Windows 系统的强大 shell 和脚本语言，支持对象管道和 .NET 框架；
- 其他Shell，还有许多其他的 shell，如 Ash、Dash、Tcsh 等，每个 shell 都有自己的特点和用途；

### 3、shell的类型

在 Linux 系统中，shell 的类型可以根据交互方式分为交互式 shell 和非交互式 shell

1. 交互式 shell
   - 用户可以直接在终端与 shell 进行交互；
   - 用户输入命令，shell 执行并返回结果；
   - 用户可以实时输入命令、查看输出，并于 shell 进行交互；
   - 交互式 shell 通常用于用户直接操作系统，执行命令、运行程序等；

2. 非交互式 shell
   - 用户通过脚本或命令文件来执行一系列的命令，而不需要实时的交互；
   - 用户将命令写入脚本文件中，然后通过 shell 执行脚本文件；
   - 执行过程中不需要用户的实时输入和交互；
   - 非交互式 shell 通常用于批处理、自动化任务、定时任务等；

在实际使用中，交互式 shell 和非交互式 shell 可以根据需要灵活选择。交互式 shell 适合用户需要实时操作和交互的场景，非交互式 shell 适合批量处理和自动化任务的场景。

### 4、shell的启动方式

在 Linux 系统中，shell 可以通过以下几种方式启动：

1. 登录时启动：当用户登录到系统时，系统会自动启动一个交互式 shell 作为用户的默认 shell，这通常是通过登录管理器（getty、login等）来实现的，用户可以直接在终端中输入命令与 shell 进行交互；

2. 执行脚本文件：用户可以编写一个包含一系列命令的脚本文件，通过在终端中执行脚本文件，可以启动相应的shell，并按照脚本中的命令顺序执行；

3. 运行shell命令：用户可以直接在终端中输入 shell 命令，并按下回车键执行，系统将启动一个临时的 shell 实例来执行该命令，并在执行完毕后退出；

4. 后台运行：用户可以使用特殊的符号（如：&）将命令放在后台运行，这样命令将在新的 shell 实例中以后台进程的方式运行，用户可以继续在终端中输入其他命令，而不必等待后台命令执行完毕；

   