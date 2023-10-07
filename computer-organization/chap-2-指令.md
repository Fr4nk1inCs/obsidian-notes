#计算机 #底层 #硬件 

# 引言
- 指令 (instruction): 计算机语言中的 "单词"
- 指令集/指令系统 (instruction set): 计算机语言中的 "词汇表"
- 指令集架构 (ISA, Instruction Set Architecture)
	- 指令集
	- 指令集编码
	- 基本数据类型
- 常见的指令集
	- x86: Intel, AMD
	- MIPS: 龙芯
	- ARM: Apple, 华为
	- RISC-V: 教学用途
		- 开源
		- 模块化
		- 可拓展
		- 成熟
	- LC-3: 教学用途
		- 简单

- RISC-V 指令集
	- RV32 操作数为 32 位
	- RV64 操作数为 64 位

RISC-V 操作数

|       名字        |                                示例                                |                                                       注解                                                        |
|:-----------------:|:------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------:|
|    32个寄存器     |                             `x0`~`x31`                             |                           快速定位数据. 在 RISC-V 中, 只对在寄存器中的数据执行算术运算                            |
| $2^{61}$ 个存储字 | `Memory[0]`,`Memory[8]`, ..., `Memory[18,446,744,073,709,551,608]` | 只能被数据传输指令访问. RISC-V 使用字节寻址, 因此顺序双字访问相差 8. 存储器保存数据结构, 数组和换出的寄存器的内容 |

RISC-V 汇编语言: 主要学习 RV32 指令集
- 共 47 条
- 主要学习其中的 37 条



# 硬件设计三条基本原则
- 设计原则 1: 简单源于规整
	- 规则性使实现更简单.
	- 简单性可以以更低的成本实现更高的性能.
	- 例如: 要求每条指令恰好有三个操作数, 使得硬件简单.
- 设计原则 2: 更少则更快 ^accfb7
	- 数量过多的寄存器可能会增加时钟周期, 因为电信号传输的距离越远, 所花费的事件就更长.
- 设计原则 3: 优秀的设计需要折中 ^64cd9b
	- RISC-V 设计人员选择的折中方案是保持所有指令长度相同, 对于不同的指令使用不同的指令格式.

# 计算机硬件的操作数
- 位 (bit): 最小单位
- 字节 (byte): 编址的最小单位
- 字 (word): 32 bit, RV32 的基本操作单元
- 半字 (halfword): 16 bit, 两个字节, 不是任意组合
- 双字 (double word): 64 bit, RV64 的基本单元
- 寄存器操作数
	- 算术运算操作数的主要来源
	- 32 个, 表示为 `x0` ~ `x31`
	- 限制为 32 个: [[#^accfb7|设计原则 2 - 更少则更快]]
	- [[RISC-V-MISC#寄存器约定|寄存器约定]]

- 存储器操作数
	- 存储复合数据
		- 数组
		- 结构体
		- 动态数据
	- 以字节为单位进行编址 (历史原因)
	- 大小端问题
		- 大端 Big Endian:  使用最左边或 "大端" 字节的地址作为双字地址
		- 小端 Little Endian: 使用最右边或 "小端" 字节的地址作为双字地址
		- 例如: `x1001` 和 `x1000` 组成一个双字节, 其大端字节为 `x1001`, 小端字节为 `x1000`
		- RISC-V 使用的是小端模式
	- 对齐问题
		- 一个字占用 4 个字节, 是否必须为 4 的倍数? 
			- 不同于有些架构,  RISC-V 不要求必须对齐
	- 与寄存器进行比较
		- 寄存器访问速度比存储器快
		- 存储器中的数据操作需要使用 Load, Store 操作
			- 用到更多指令
		- 寄存器可以同时操作三个数据
	
		> 编译器必须尽可能多地使用变量寄存器, 仅将不太常用的变量存放到内存中, 寄存器优化很重要!

- 常数/立即数操作数
	- 指令中指定的常量数据
	- 指令的后缀为 `i`
	- [[chap-1-计算机抽象及相关技术#加速经常性事件|加速经常性事件]]
		- 小常数很常见
		- 立即操作数避免加载指令
		- 节省了宝贵的寄存器资源

# 有符号数与无符号数
- MSB, most significant bit, 最高有效位
- LSB, least significant bit, 最低有效位
- 原码: $x = (-1)^{\text{MSB}}\left(\sum_{i=0}^{l-2} x_i\times2^i\right)$
- 补码: $x = \text{MSB}\times(-2^{l-1})+\sum_{i=1}x_i\times 2^i$
	- 补码求相反数: 取反加一
- 符号位拓展
	- 有符号拓展 (sign-extend, SEXT): 将 MSB 复制并填充到数据的左侧  ^a0a00e
		- `1110 => 1111 1110` 
		- `0001 => 0000 0001`
	- 无符号拓展 (zero-extend, ZEXT): 将 0 填充到数据的左侧 ^e4d720
		- `1110 => 0000 1110`

# 计算机中的指令表示
- [[RISC-V-MISC#RISC-V 指令集|RISC-V 指令集]]
	- 32 bit
	- 划分为多个字段
	- 用字段来表示操作类型, 寄存器编号等信息
- 指令格式: [[#^64cd9b|折中]]

|            指令格式            |       类型        |       例子        |
|:------------------------------:|:-----------------:|:-----------------:|
| [[RISC-V-MISC#R-type\|R-type]] |    寄存器类型     |       `add`       |
| [[RISC-V-MISC#I-type\|I-type]] | 短立即数/加载类型 | `addi`, `lw`/`ld` |
| [[RISC-V-MISC#S-type\|S-type]] |     存储类型      |     `sw`/`sd`     |
| [[RISC-V-MISC#B-type\|B-type]] |   条件跳转类型    |       `beq`       |
| [[RISC-V-MISC#J-type\|J-type]] |  无条件跳转类型   |       `jal`       |
| [[RISC-V-MISC#U-type\|U-type]] |   长立即数类型    |       `lui`       |

^1f45d3

- RISC-V 字段命名 ^c68e3c
	- opcode: 操作码
	- funct: 另外的操作码字段
	- rd: 目的寄存器操作数, 用来存放结果
	- rs(1/2): (第 1/2 个) 源操作数寄存器
	- immediate: 立即数
	- offset: 偏移量

> 操作码占 7 位, 寄存器占 5 位

---
- 当代计算机构建基于两个关键原则
	- 指令由数字形式表示
	- 指令和数据一样保存在存储器中读写
- 引出程序存储的概念
- 二进制兼容性
	- 程序以二进制形式发布
	- 促使指令集生态的形成
 
# 汇编与指令
## 算术操作
- `add`: 加
	- 汇编语言: `add rd, rs1, rs2` // `rd = rs1 + rs2`
	- 指令: [[RISC-V-MISC#^7e851f|R-type]] `0000000|xxxxx|xxxxx|000|xxxxx|0110011`
		- `add x9, x20, x21`→`00000001010110100000010010110011`/`0x015A04B3`
- `sub`: 减
	- 汇编语言: `sub rd, rs1, rs2` // `rd = rs1 - rs2`
	- 指令: [[RISC-V-MISC#^7e851f|R-type]] `0100000|xxxxx|xxxxx|000|xxxxx|0110011`
		- `sub x9, x20, x21`→`01000001010110100000010010110011`/`0x415A04B3`
- 例子: `f = (g + h) - (i + j)`
	- `f`, `g`, …, `j` 在 `x19`, `x20`, …, `x23` 中
 ```riscvasm
add x5, x20, x21
add x6, x22, x23
sub x19, x5, x6
```
- `addi`: 立即数加 ^6d4d88
	- 汇编语言: `addi rd, rs, imm12` // `rd = rs + imm12`
	- 指令: [[RISC-V-MISC#^046c68|I-type]] `xxxxxxxxxxxx|xxxxx|000|xxxxx|0010011`
		- `addi x22, x22, 4`→`00000000010011000000110000010011`/`0x004C0C13`

## 访存操作
- RV64I
	- 取双字: `ld rd, offset(rs)`
		- 例如: `ld x9, 8(x22)` `x9 = x22[1]`
	- 存双字: `sd rs2, offset(rs1)`
- 加载字节/半字/字 ([[#^a0a00e|SEXT]]) (8/16/32 bit)
	- 指令格式: [[RISC-V-MISC#^7fdca2|I-type]]
	- 字节: `lb rd, offset(rs1)` ^1e598d
	- 半字: `lh rd, offset(rs1)`
	- 字: `lw rd, offset(rs1)`
- 加载字节/半字/字 ([[#^e4d720|ZEXT]])
	- 指令格式: [[RISC-V-MISC#^7fdca2|I-type]]
	- 字节: `lbu rd, offset(rs1)`
	- 半字: `lhu rd, offset(rs1)`
- 加载大立即数
	- `lui rd, imm20` // `rd = SEXT(imm20 << 12)`
	- 与 [[#^6d4d88|addi]] 指令搭配课加载 32 位立即数
	- [[RISC-V-MISC#U-type|U-type]]
- 存储字节/半字/字
	- 指令格式: [[RISC-V-MISC#^873030|S-type]]
	- 字节: `sb rd, offset(rs1)` ^61c903
	- 半字: `sh rd, offset(rs1)`
	- 字: `sw rd, offset(rs1)`

## 逻辑操作
- 位移操作
	- 寄存器位移
		- 指令格式: [[RISC-V-MISC#^7e851f|R-type]]
		- 逻辑左移: `sll rd, rs1, rs2` // `rd = rs1 << rs2`
		- 逻辑右移: `srl rd, rs1, rs2` //  `rd = rs1 >> rs2`
		- 算术右移: `sra rd, rs1, rs2` //  `rd = rs1 >> rs2`
	- 立即数位移:
		- 指令格式: [[RISC-V-MISC#^36ca7b|I-type]]
		- 逻辑左移: `sll rd, rs1, shamt` // `rd = rs1 << shamt`
		- 逻辑右移: `srl rd, rs1, shamt` // `rd = rs1 >> shamt`
		- 算术右移: `sra rd, rs1, shamt` // `rd = rs1 >> shamt`
- 逻辑运算
	- 寄存器
		- 指令格式: [[RISC-V-MISC#^7e851f|R-type]]
		- 按位与: `and rd, rs1, rs2` // `rd = rs1 & rs2`
		- 按位或: `or rd, rs1, rs2` // `rd = rs1 | rs2`
		- 按位异或: `xor rd, rs1, rs2` // `rd = rs1 ^ rs2`
			- 可以用来实现按位非: `xor rd, rs, rs` // `rd = ~rs`
	- 立即数
		- 指令格式: [[RISC-V-MISC#^046c68|I-type]]
		- 按位与: `andi rd, rs1, imm12` // `rd = rs1 & SEXT(imm12)`
		- 按位或: `ori rd, rs1, imm12` // `rd = rs1 | SEXT(imm12)`
		- 按位异或: `xori rd, rs1, imm12` // `rd = rs1 ^ SEXT(imm12)`

## 条件分支
- 指令格式: [[RISC-V-MISC#B-type|B-type]]
- 如果符合条件, 则跳转到对应的标签, 否则继续执行下一条指令.

> [[RISC-V-MISC#RISC-V 指令集]]


| 指令                 | 名称                    | 操作                            |
| -------------------- | ----------------------- | ------------------------------- |
| `beq x5, x6, label`  | 相等即跳转              | `if (x5 == x6) goto PC + label` |
| `bne x5, x6, label`  | 不等即跳转              | `if (x5 != x6) goto PC + label` |
| `blt x5, x6, label`  | 小于即跳转              | `if (x5 < x6) goto PC + label`  |
| `bge x5, x6, label`  | 大于等于即跳转          | `if (x5 >= x6) goto PC + label` |
| `bltu x5, x6, label` | 小于即跳转 (无符号)     | `if (x5 < x6) goto PC + label`  |
| `bgeu x5, x6, label` | 大于等于即跳转 (无符号) | `if (x5 >= x6) goto PC + label` |

### 示例
#### `if` 条件语句
```c
if (i == j) 
	f = g + h;
else 
	f = g - h;
```
```riscvasm
	bne x22, x23, Else
	add x19, x20, x21
	beq x0,  x0,  Exit // unconditional

Else:
	sub x19, x20, x21

Exit: 
	...
```
#### `while` 循环
```c
while (save[i] == k)  // k in x24, save.addr in x25
	i++;              // i in x22
```
```riscvasm
Loop:
	slli x10, x22, 3   // x10 = 8 * i
	add  x10, x10, x25 // x10 is the addr of save[i]
	ld   x9,  0(x10)   // x9 = save[i]
	bne  x9,  x24, Exit
	addi x22, x22, i   // i++
	beq  x0,  x0,  Loop

Exit:
	...
```

#### `switch`/`case` 语句
- 分支地址表, 包含代码中标签对应地址的一个数组
- `jalr`: 间接跳转指令
	- e.g.: 中断向量表

## 无条件跳转
- `jal`: `jal rd, label` // `rd = pc + 4; pc += SEXT(offset)` ^4b2cfe
	- 把下一条指令的地址 (RV32 为 `PC+4`, RV64 为 `PC+8`) 保存到 `rd` 中
	- 将 `pc` 设为 `label` 的地址
	- [[RISC-V-MISC#J-type|J-type]]
- `jalr`: `jalr rd, offset(rs1)` // `rd = pc+4; pc = (rs1 + SEXT(offset))&~1` ^6adf2e
	- 把下一条指令的地址 (RV32 为 `PC+4`, RV64 为 `PC+8`) 保存到 `rd` 中
	- 将 `pc` 设为 `rs1 + SEXT(offset)`, 最低位设为 0
	- [[RISC-V-MISC#I-type|I-type]]

# 计算机硬件对过程 (函数) 的支持
- caller: 调用者, 启动过程, 并提供必要参数值的程序
- callee: 被调用者, 根据调用者提供的参数，执行一系列已存储的指令的过程，然后将控制权返还给调用者
- `pc`: 程序计数器, program counter, 一个包含程序中正在执行指令的地址的寄存器
- 过程调用相关指令
	- [[#^4b2cfe|jal]]: jump and link
		- 一般用于过程调用 `jal x1, ProcedureLabel`
	- [[#^6adf2e|jalr]]: jump and link register
		- 一般用于过程调用结束后的返回 `jalr x0, 0(x1)` (`x0` 无法被改变, 所以 `pc+4` 被丢弃)
		- 也可以用于 `switch`/`case` 语句

## 使用更多的寄存器
- 过程调用会遇到参数传递的问题
	- 传递的参数是否有数量限制? 参数多了如何处理?
	- 子过程是否可以毫无顾忌的使用所有寄存器?
	- 过程返回后如何恢复现场?
- 解决思路: 充分利用大容量的主存
	- 参数传递采用传送地址的方式
	- 将寄存器换出到存储器中
- 解决方案: 栈 stack
	- LIFO 的数据结构
	- 栈指针, stack pointer, sp: 存放栈中最新分配的地址
		- RISC-V 中约定使用寄存器 `x2` 存放 sp
- [[RISC-V-MISC#寄存器约定|寄存器使用约定]]
	- `x5`-`x7`, `x28`-`x31`: 临时寄存器, 在调用过程中不被 callee 保存
	- `x8`-`x9`, `x18`-`x27`: 保存寄存器, 在调用过程中必须被保存, 一旦被 callee 使用, 由 callee 保存并恢复
	- 在过程调用中需要/不需要保存的寄存器![[Pasted image 20220322144606.png|500]]
## 过程
- 叶子过程 leaf procedures
	- 不调用其他过程的过程, 称为叶子过程
- 嵌套过程 nested procedures
	- 递归 Recursive: 直接或间接地调用自己的过程
	- caller 做的事
		- 调用前, 保存返回地址, 保存所有的参数和返回后需要用到的临时变量
		- 调用后, 从栈中恢复现场
## 内存空间分配
- 在栈中为新数据分配空间
	- 过程帧/活动记录: 栈中包含过程保存的寄存器和局部变量的段.
	- 过程帧的地址由帧指针 frame pointer, fp 确定, RISC-V 约定 `x8` 存放 fp
	- 在过程中, fp 比 sp 更稳定, 方便作为基地址

	![[Pasted image 20220322145929.png|500]]

![[Pasted image 20220322150729.png|inlineR|300]]
- 内存布局
	- 保留区域
	- 代码段
	- 静态数据/全局变量
		- `static`
		- constant arrays and strings
		- `x3`/gp 为基址进行寻址的数据
	- 动态数据/堆
		- C: `malloc`, C++/Java: `new`
	- 栈
- 堆与栈相向而长, 以达到对内存的高效利用

## 例子
- 叶子过程

```c
long long int leaf(
	long long int g,  // in x10
	long long int h,  // in x11
	long long int i,  // in x12
	long long int j)  // in x13
{
	long long int f;  // in x20
	f = (g + h) - (i + j); // temporaries in x5, x6
	return f;
}
```
```riscvasm
leaf:
	addi sp,  sp, -24 // push x5, x6, x20 on stack
	sd   x5,  16(sp)
	sd   x6,  8(sp)
	sd   x20, 0(sp)
	add  x5,  x10, x11 // x5 = g + h
	add  x6,  x12, x13 // x6 = i + j
	sub  x20, x5,  x6  // f = x5 - x6
	addi x10, x20, 0   // copy f to return reg
	ld   x20, 0(sp)    // pop x5, x6, x20 from stack
	ld   x6,  8(sp)
	ld   x5,  16(sp)
	addi sp,  sp, 24
	jalr x0,  0(x1)    // return
```

- 嵌套过程

```c
long long int fact(long long int n) // in x10
{
	if (n < 1) return 1;
	else return n * fact(n - 1); // return in x10
}
```
```riscvasm
// suppose we have a instruction mul to multiply two reg
fact:
	addi sp,  sp,  -16 // push n and return address
	sd   x1,  8(sp)    // return addr
	sd   x10, 0(sp)    // n
	addi x5,  x10, -1
	bge  x5,  x0,  L1  // if n >= 1, goto L1
	addi x10, x0,  1   // else return 1
	addi sp,  sp,  16  // pop
	jalr x0,  0(x1)
L1:
	addi x10, x10, -1  // n = n - 1
	jal  x1,  fact     // call fact(n - 1)
	addi x6,  x10, 0   // x6 <- fact(n - 1)
	ld   x10, 0(sp)    // restore n
	ld   x1,  8(sp)    // restore return addr
	addi sp,  sp,  16  // pop
	mul  x10, x10, x6  // return n * fact(n-1)
	jalr x0, 0(x1)
```

# 人机交互
- ASCII 码
	- 8 bit per char
	- 对字节进行操作, [[#^1e598d|lb]]/[[#^61c903|sb]] 指令
- 字符串: 以 ASCII 值为 0 的符号结尾

## 例子: 字符串拷贝
```c
void strcpy(char dest[], char src[])
{
	size_t i;
	i = 0;
	while ((dest[i] = src[i]) != '\0')
		i++;
}
```
```riscvasm
strcpy:
	addi sp,  sp,  -8  // push temp reg
	sd   x19, 0(sp)
	add  x19, x0,  x0  // i = 0
loop:
	add  x5,  x19, x10 // x5 = src + i (addr. src[i])
	lbu  x6,  0(x5)    // x6 = src[i]
	add  x7,  x19, x11 // x7 = dest + i
	sbu  x6,  0(x7)
	beq  x6,  x0,  ret // if src[i] == 0 return
	addi x19, x19, 1
	jal  x0,  loop
ret:
	ld   x19, 0(sp)    // pop
	addi sp,  sp,  8
	jalr x0,  0(x1)
```

# 对大立即数的编址和寻址
- 如何向寄存器中写入一个 32 bit 的数据
	- load/store 指令
	- `addi`+`slli` 指令组合 (不断运算+左移)
	- `jal` 指令可生成 20 bit 立即数, 但只能影响 `pc`
- `lui` 指令, [[RISC-V-MISC#U-type|U-type]]
	- `lui` + `addi` 可以直接写入一个 32 bit 的数据, 在汇编中可以直接用 `li` 写入 32 bit 数据, 由汇编器翻译为两条指令.


		```riscvasm
		lui  x5, 0x12345
		addi x5, x5, 0x678 // x5 = 0x12345678
		```
	
| 指令         | 示例              | 含义              | 注释                         |
| ------------ | ----------------- | ----------------- | ---------------------------- |
| 取立即数高位 | `lui x5, 0x12345` | `x5 = 0x12345000` | 取右移 12 位后的 20 位立即数 | 


- PC 相对寻址
	- 一种寻址方式, 地址为 PC 和指令中的常量之和
	- [[RISC-V-MISC#B-type|B-type]]: 条件分支, 可以跳转到前后 $\pm 2^{12}$ 字节 ($\pm$ 4KByte)
	- [[RISC-V-MISC#J-type|J-type]]: 无条件跳转, 可以跳转到前后 $\pm 2^{20}$ 字节 ($\pm$ 1MByte)
- 长距离跳转: `lui` + `jalr`

![[Pasted image 20220322161034.png]]

# 指令与并行性: 同步
- 数据竞争 data race
	- 如果来自两个不同的线程的访存请求访问同一个位置, 至少有一个是写, 且连续出现, 那么这两次存储访问形成了数据竞争
	- 解决办法：通过加锁 lock 和解锁 unlock 同步操作创建只有单个处理器(或进程)可以操作的区域 (互斥区)
	- 在硬件上，通过原子交换原语来实现
- RISC-V 相关指令: `lr.d`, `sc.d` 
  
# 以 C 排序为例
```c
void swap(long long int v[], int k)
{
	long long int temp;
	temp = v[k];
	v[k] = v[k + 1];
	v[k + 1] = v[k];
}
```
![[Pasted image 20220322161755.png|400]]

```c
void sort(long long int v[], size_t int n)
{
	size_t i, j;
	for (i = 0; i < n; i++)
		for (j = i - 1; j >= 0 && v[j] > v[j+1]; j--)
			swap(v, j);
}
```
![[Pasted image 20220322162013.png|600]]
![[Pasted image 20220322162018.png|600]]
![[Pasted image 20220322162025.png|600]]

# 谬误与陷阱
- 谬误 1: 强大的指令意味着更高的性能
- 谬误 2: 用汇编语言编程以获得最高的性能
- 谬误 3: 商用计算机二进制兼容的重要性意味着成功的指令系统无须改变
- 陷阱 1: 在字节寻址的机器中, 连续的字或双字地址相差不为 1
- 陷阱 2: 在变量的定义过程外, 不要将指针指向该变量