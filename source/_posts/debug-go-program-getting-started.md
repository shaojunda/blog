---
title: debug Go 程序入门介绍之 Delve 篇
date: 2021-05-23 17:32:47
tags:
  - golang
  - programing
  - dlv
keywords:
  - golang
  - dlv
  - go tool compile
  - go tool objdump

description: debug golang program getting started
---

今天简单了解了一下 [Delve](https://github.com/go-delve/delve) 这个 go 语言调试工具，和两个 go 编译和反编译命令 `go tool compile` 和 `go tool objdump` 还是挺有意思的。这篇文章会介绍一下 Delve 的使用方式。(以下内容的运行环境均为 Linux)

## Delve

安装方式请参考 [installation](https://github.com/go-delve/delve/tree/master/Documentation/installation)

先以 `hello world` 程序为例我们看一下 dlv 的使用

```golang
// hello.go
package main

func main() {
	println("hello world")
}
```

执行 `go build hello.go` 编译 `hello.go` 文件得到可执行文件 `hello`。

执行 `readelf -h ./hello` 可得到
```ssh
[root@4158ddf6b44d home]# readelf -h ./hello
ELF Header:
Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
Class:                             ELF64
Data:                              2's complement, little endian
Version:                           1 (current)
OS/ABI:                            UNIX - System V
ABI Version:                       0
Type:                              EXEC (Executable file)
Machine:                           Advanced Micro Devices X86-64
Version:                           0x1
Entry point address:               0x455780
Start of program headers:          64 (bytes into file)
Start of section headers:          456 (bytes into file)
Flags:                             0x0
Size of this header:               64 (bytes)
Size of program headers:           56 (bytes)
Number of program headers:         7
Size of section headers:           64 (bytes)
Number of section headers:         25
Section header string table index: 3
```

可以看到 `Entry point address: 0x455780` 这个就是 `hello.go` 程序的执行入口。接下来使用 `dlv` 对 `hello.go` 程序进行调试。

执行 `dlv exec ./hello` 会进入到调试模式
<!--more-->
```
[root@4158ddf6b44d home]# dlv exec ./hello
Type 'help' for list of commands.
(dlv)
```

输入 `help` 可以看到可用的命令列表
```
(dlv) help
The following commands are available:

Running the program:
    call ------------------------ Resumes process, injecting a function call (EXPERIMENTAL!!!)
    continue (alias: c) --------- Run until breakpoint or program termination.
    next (alias: n) ------------- Step over to next source line.
    restart (alias: r) ---------- Restart process.
    step (alias: s) ------------- Single step through program.
    step-instruction (alias: si)  Single step a single cpu instruction.
    stepout (alias: so) --------- Step out of the current function.

Manipulating breakpoints:
    break (alias: b) ------- Sets a breakpoint.
    breakpoints (alias: bp)  Print out info for active breakpoints.
    clear ------------------ Deletes breakpoint.
    clearall --------------- Deletes multiple breakpoints.
    condition (alias: cond)  Set breakpoint condition.
    on --------------------- Executes a command when a breakpoint is hit.
    trace (alias: t) ------- Set tracepoint.

Viewing program variables and memory:
    args ----------------- Print function arguments.
    display -------------- Print value of an expression every time the program stops.
    examinemem (alias: x)  Examine memory:
    locals --------------- Print local variables.
    print (alias: p) ----- Evaluate an expression.
    regs ----------------- Print contents of CPU registers.
    set ------------------ Changes the value of a variable.
    vars ----------------- Print package variables.
    whatis --------------- Prints type of an expression.

Listing and switching between threads and goroutines:
    goroutine (alias: gr) -- Shows or changes current goroutine
    goroutines (alias: grs)  List program goroutines.
    thread (alias: tr) ----- Switch to the specified thread.
    threads ---------------- Print out info for every traced thread.

Viewing the call stack and selecting frames:
    deferred --------- Executes command in the context of a deferred call.
    down ------------- Move the current frame down.
    frame ------------ Set the current frame, or execute command on a different frame.
    stack (alias: bt)  Print stack trace.
    up --------------- Move the current frame up.

Other commands:
    config --------------------- Changes configuration parameters.
    disassemble (alias: disass)  Disassembler.
    edit (alias: ed) ----------- Open where you are in $DELVE_EDITOR or $EDITOR
    exit (alias: quit | q) ----- Exit the debugger.
    funcs ---------------------- Print list of functions.
    help (alias: h) ------------ Prints the help message.
    libraries ------------------ List loaded dynamic libraries
    list (alias: ls | l) ------- Show source code.
    source --------------------- Executes a file containing a list of delve commands
    sources -------------------- Print list of source files.
    types ---------------------- Print list of types

Type help followed by a command for full documentation.
```

这里我主要使用 `b`, `c`, `si`, `bt` 来感受以下 `dlv` 的使用。

首先输入 `b *0x455780` 在程序入口处打一个断点
```
(dlv) b *0x455780
Breakpoint 1 set at 0x455780 for _rt0_amd64_linux() /usr/lib/golang/src/runtime/rt0_linux_amd64.s:8
```

结下来输入 `c`
```
(dlv) c
> _rt0_amd64_linux() /usr/lib/golang/src/runtime/rt0_linux_amd64.s:8 (hits total:1) (PC: 0x455780)
Warning: debugging optimized function
     3: // license that can be found in the LICENSE file.
     4:
     5: #include "textflag.h"
     6:
     7: TEXT _rt0_amd64_linux(SB),NOSPLIT,$-8
=>   8:         JMP     _rt0_amd64(SB)
     9:
    10: TEXT _rt0_amd64_linux_lib(SB),NOSPLIT,$0
    11:         JMP     _rt0_amd64_lib(SB)
(dlv)
```

可以看出执行到了 `JMP     _rt0_amd64(SB)` 这条汇编指令。我们可以输入 `si` 来继续执行下一条汇编指令
```
(dlv) si
> _rt0_amd64() /usr/lib/golang/src/runtime/asm_amd64.s:15 (PC: 0x451bd0)
Warning: debugging optimized function
    10: // _rt0_amd64 is common startup code for most amd64 systems when using
    11: // internal linking. This is the entry point for the program from the
    12: // kernel for an ordinary -buildmode=exe program. The stack holds the
    13: // number of arguments and the C-style argv.
    14: TEXT _rt0_amd64(SB),NOSPLIT,$-8
=>  15:         MOVQ    0(SP), DI       // argc
    16:         LEAQ    8(SP), SI       // argv
    17:         JMP     runtime·rt0_go(SB)
    18:
    19: // main is common startup code for most amd64 systems when using
    20: // external linking. The C startup code will call the symbol "main"
(dlv)
```
这样我们就进入到了 `rt0_amd64` 内部，继续执行三次 `si` 可以看到程序进入到了 `runtime·rt0_go`
```
(dlv) si
> runtime.rt0_go() /usr/lib/golang/src/runtime/asm_amd64.s:89 (PC: 0x451be0)
Warning: debugging optimized function
    84: DATA _rt0_amd64_lib_argv<>(SB)/8, $0
    85: GLOBL _rt0_amd64_lib_argv<>(SB),NOPTR, $8
    86:
    87: TEXT runtime·rt0_go(SB),NOSPLIT,$0
    88:         // copy arguments forward on an even stack
=>  89:         MOVQ    DI, AX          // argc
    90:         MOVQ    SI, BX          // argv
    91:         SUBQ    $(4*8+7), SP            // 2args 2auto
    92:         ANDQ    $~15, SP
    93:         MOVQ    AX, 16(SP)
    94:         MOVQ    BX, 24(SP)
(dlv)
```
在 `runtime·rt0_go` 中进行诸如初始化等操后会执行 `hello.go` 的 `main` 函数。


## 调试向已关闭的 channel 写数据
```golang
// test1.go
package main

func main() {
	ch := make(chan int)
	close(ch)
	ch <- 1
}
```
对于这段程序我们运行 `go run test1.go` 会得到：
```
[root@4158ddf6b44d home]# go run test.go
panic: send on closed channel

goroutine 1 [running]:
main.main()
        /home/test.go:6 +0x63
exit status 2
```

这时我们可以使用 dlv 来查看具体是在哪一行 panic 的。

```
[root@4158ddf6b44d home]# dlv exec ./test
Type 'help' for list of commands.
(dlv) c
> [unrecovered-panic] runtime.fatalpanic() /usr/lib/golang/src/runtime/panic.go:1189 (hits goroutine(1):1 total:1) (PC: 0x429e30)
Warning: debugging optimized function
        runtime.curg._panic.arg: interface {}(string) "send on closed channel"
  1184: // fatalpanic implements an unrecoverable panic. It is like fatalthrow, except
  1185: // that if msgs != nil, fatalpanic also prints panic messages and decrements
  1186: // runningPanicDefers once main is blocked from exiting.
  1187: //
  1188: //go:nosplit
=>1189: func fatalpanic(msgs *_panic) {
  1190:         pc := getcallerpc()
  1191:         sp := getcallersp()
  1192:         gp := getg()
  1193:         var docrash bool
  1194:         // Switch to the system stack to avoid any stack growth, which
(dlv)
```
在这里输入 `bt` 可以查看调用栈
```
(dlv) bt
0  0x0000000000429e30 in runtime.fatalpanic
   at /usr/lib/golang/src/runtime/panic.go:1189
1  0x00000000004298ad in runtime.gopanic
   at /usr/lib/golang/src/runtime/panic.go:1064
2  0x00000000004041b3 in runtime.chansend
   at /usr/lib/golang/src/runtime/chan.go:187
3  0x0000000000403bc5 in runtime.chansend1
   at /usr/lib/golang/src/runtime/chan.go:127
4  0x00000000004587d3 in main.main
   at ./test.go:6
5  0x000000000042c2ba in runtime.main
   at /usr/lib/golang/src/runtime/proc.go:203
6  0x0000000000453d91 in runtime.goexit
   at /usr/lib/golang/src/runtime/asm_amd64.s:1373
(dlv)
```
这里可以看到是 `runtime.chansend` panic 了，这是可以输入 `up 2` 可以看到 panic 的具体位置
```
(dlv) up 2
> [unrecovered-panic] runtime.fatalpanic() /usr/lib/golang/src/runtime/panic.go:1189 (hits goroutine(1):1 total:1) (PC: 0x429e30)
Warning: debugging optimized function
Frame 2: /usr/lib/golang/src/runtime/chan.go:187 (PC: 4041b3)
   182:
   183:         lock(&c.lock)
   184:
   185:         if c.closed != 0 {
   186:                 unlock(&c.lock)
=> 187:                 panic(plainError("send on closed channel"))
   188:         }
   189:
   190:         if sg := c.recvq.dequeue(); sg != nil {
   191:                 // Found a waiting receiver. We pass the value we want to send
   192:                 // directly to the receiver, bypassing the channel buffer (if any).
(dlv)
```
再次输入两次 `up` 得到如下输出：
```
(dlv) up
> [unrecovered-panic] runtime.fatalpanic() /usr/lib/golang/src/runtime/panic.go:1189 (hits goroutine(1):1 total:1) (PC: 0x429e30)
Warning: debugging optimized function
Frame 3: /usr/lib/golang/src/runtime/chan.go:127 (PC: 403bc5)
   122: }
   123:
   124: // entry point for c <- x from compiled code
   125: //go:nosplit
   126: func chansend1(c *hchan, elem unsafe.Pointer) {
=> 127:         chansend(c, elem, true, getcallerpc())
   128: }
   129:
   130: /*
   131:  * generic single channel send/recv
   132:  * If block is not nil,
(dlv)
(dlv) up
> [unrecovered-panic] runtime.fatalpanic() /usr/lib/golang/src/runtime/panic.go:1189 (hits goroutine(1):1 total:1) (PC: 0x429e30)
Warning: debugging optimized function
Frame 4: ./test.go:6 (PC: 4587d3)
     1: package main
     2:
     3: func main() {
     4:         ch := make(chan int)
     5:         close(ch)
=>   6:         ch <- 1
     7: }
(dlv)
```
到此我们可以看到是在 `main` 函数第 6 行向 channel 里写数据导致的 panic， 具体 panic 是在 `/usr/lib/golang/src/runtime/chan.go:187` 处执行的。

## 总结

dlv 的使用还是挺简单的，目前主要用到了
* b 打断点
* c 继续执行
* bt 打印 stack trace
* si 执行一条汇编语句
* r 重新执行
* up `[<m>]` 向上移动 m 个 frame
* down `[<m>]` 向下移动 m 个 frame

