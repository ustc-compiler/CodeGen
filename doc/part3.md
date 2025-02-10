## 栈空间分配
由于LLVM IR有无限的虚拟寄存器，所以并没有显式地处理栈相关的操作，只是使用`alloca`来进行空间的分配。然而，实际体系结构中寄存器的数目是有限的，所以需要把一些寄存器放不下的内容溢出到栈上（包括数组）。
你需要在`{WorkSpace}/src/Backend/StackAlloc.cpp`中实现栈空间分配的代码，请确保完成阅读[文档](#可能用到的参考文档)后再写这部分代码，此函数以`Function*`为参数，代表待处理的函数，它的接口与你们之前实验的接口一致，通过这个指针你可以获得待处理函数的各种信息（如各个参数的信息、参数个数）。函数返回分配的总空间的大小（字节数）。

你需要注意：
- 寄存器分配产生的`reg_map`可以直接使用，它就是你在寄存器分配的时候生成的`reg_map`，你需要为其中`reg_num`为 -1 的`interval`分配栈空间，以`IR2asm::RegBase`给出其基地址，存入`stack_map`当中。
- 同时，通过栈传递的函数参数在栈上的位置（如ABI约定）还要保存到`arg_on_stack`当中，不论它是否被分配了寄存器。
- 你可以通过`bool have_temp_reg`和`bool have_func_call`来判断此函数是否使用临时寄存器以及内部是否有函数调用，它们都是`CodeGen`类的数据域。
- 类似的，你可以通过`int caller_saved_reg_num`和`int temp_reg_store_num`获得调用者保存的寄存器个数以及临时寄存器暂存空间的大小，通过`const int reg_size`来获得寄存器的字节数，它们都是`CodeGen`类的数据域并且已经在调用栈分配之前维护好。
- 注意函数形参的空间分配，有些参数通过栈传递，已经拥有了栈空间，不必重新分配。
- 局部数组并不在`reg_map`中，而是在开始块中通过`alloca`语句分配空间，所以你还需要遍历`fun->get_entry_block()->get_instructions()`，根据其中的`alloca`语句进行空间分配。
- 理论上讲你甚至可以将所有变量在寄存器分配阶段全部溢出到栈，这将使得寄存器分配的代码非常简单，但是代码将经历大量内存操作，这也属于设计选择互相影响的范畴。但是这种写法视为未完成寄存器分配。

### 思考题

完成了上述3个关卡，请你在`doc/reports/answer.md`中回答：

1. 由于指令长度有限，所以指令的立即数是有范围的。请查阅相关资料，了解在ARM 的代码生成中如何处理超范围立即数问题，并在`doc/reports/answer.md`中进行说明。
2. 在本实验框架给出的IR实现中，可以通过强制给phi指令涉及到的所有虚寄存器分配相同的物理寄存器（或栈地址）来规避phi指令移动吗，为什么？
3. 如何在寄存器分配的时候结合函数调用ABI以及Phi指令的要求产生更高效率的代码？

### 可能用到的参考文档
- 寄存器分配数据结构、接口说明文档在`doc/RegAlloc.md`
完成第一关寄存器分配所必要的接口和数据结构说明。

- 汇编指令相关数据结构定义接口文档在`doc/AsmValue.md`。
在后端中，所有的位置（寄存器、立即数、栈地址等）使用本实验软件包提供的接口进行表示。

- 栈帧结构与ABI约定相关文档在`doc/StackFrameAndABI.md`。
在第三关栈空间分配实验中你需要按照约定进行栈空间分配。

### 评测说明

在完成本关后，你可以在平台点击评测按钮进行评测。评测结束后会在项目根目录创建`test.log`，包含每个测例是否通过的信息。评测时使用交叉编译器`arm-linux-gnueabihf-gcc`以及`qemu`在x86平台上编译并执行`compiler`输出的arm汇编码。

若你想在本地使用交叉编译器并使用`qemu`模拟arm执行，可以按以下步骤（以`ubuntu22.04`为例）：

1. 安装交叉编译工具链和 QEMU
```bash
sudo apt update
sudo apt install gcc-arm-linux-gnueabihf binutils-arm-linux-gnueabihf qemu-user qemu-user-static
```
2. 交叉编译。
假设`compiler`输出的文件是`test.s`，并且当前在项目根目录：
```bash
arm-linux-gnueabihf-gcc test.s lib/lib.c -o test.out
```
3. 使用`qemu`模拟arm环境执行
```bash
qemu-arm -L /usr/arm-linux-gnueabihf ./test.out
```


