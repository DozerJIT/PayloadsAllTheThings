# Windbg 动态调试总结S

> 本文章总结了部分windbg相关动态调试命令和技巧

## 插件

* mex

* dbgkit

* swishdbgext
* windbg-extensions

## 命令

* .load
  * !load 

* lm list model

* .hh lm   显示帮助

* !address

* r 查看寄存器

* k 查看栈

* kp
  * kb
  * kv

* d 显示

* masm/C++
  * .expr
  * .expr /s c++
  * @@C++()

* dt display type 显示type

* bp 
  * bd
  * be
  * bc
  * bp 条件断点  bp address ".if"
  * bu 延时断点
  * bm 在符号下断点 可以加通配符
* ba breakpointer access 数据访问断点 硬件断点
* u 反汇编 
  * u . 反汇编当前地址
  * ub 返回上面
  * uf 
* g
  * gu
  * ga 设置类型
  * gc 条件断点 时候使用
  * g address 执行到此地址为止 "单次断点"
* p f8
  * pc
* t f7 步入 
  * tc 步入到下个函数调用 t call 

* .if 
  * .else
  * j () command 
* | 进程切换
  * l[pid]s
* ~ 线程
  * ~[tid]s
  * ~*e k
* s 内存搜索
* e
  * eza
  * ezu
  * ea
  * eu
* .writemem address lenth

## 高级用法？

### Debug Data Model

* dx 
  * -g 
  * -r 
  * linq 语法
  * dx @$my = "hello" 定义变量
  * dx @vars.Clear()
  * $curframe current 
  * $curTread
  * $curProcess
  * $curStack
  * $cursession
  * 脚本
  * ECMAScript