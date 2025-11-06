# Opcode Execution
# 操作码的执行
In the previous section, we fetched our opcode and were preparing to decode which instruction it corresponds to to execute that instruction. Currently, our `tick` function looks like this:

在前面段，我们遍历我们的操作码，正准备解码的指令对应到执行的那个指令。当前，我们的`tick`函数看起来像这样:

```rust
pub fn tick(&mut self) {
    // Fetch
    let op = self.fetch();
    // Decode
    // Execute
}
```

This implies that decode and execute will be their own separate functions. While they could be, for Chip-8 it's easier to simply perform the operation as we determine it, rather than involving another function call. Our `tick` function thus becomes this:

这个实现的解码和执行将各自拥有分开的函数，当这样做，对于chip-8是容易执行我们所确定的操作，相当于包括另外函数的调用。我们的`tick`函数因此变成这样:

```rust
pub fn tick(&mut self) {
    // Fetch
    let op = self.fetch();
    // Decode & execute
    self.execute(op);
}

fn execute(&mut self, op: u16) {
    // TODO
}
```

Our next step is to *decode*, or determine exactly which operation we're dealing with. The [Chip-8 opcode cheatsheet](#ot) has all of the available opcodes, how to interpret their parameters, and some notes on what they mean. You will need to reference this often. For a complete emulator, each and every one of them must be implemented.

我们下一个步骤时*解码*,或者确定我们将要处理执行哪个操作。这个[chip-8解码 作弊清单]有所有的有效操作码,但是怎样解析他们的参数，和在它们上面意思的什么的一些笔记。你将经常需要引用这个。作为一个完整的模拟器，每个和它们的每一个都需要被实现。

## Pattern Matching
## 模式匹配
Fortunately, Rust has a very robust and useful pattern matching feature we can use to our advantage. However, we will need to separate out each hex "digit" before we do so.

幸运的是，Rust有一个非常强健和有用的模式匹配特性为我们所用作为我们的优势。但是，我们在此之前我们需要划分出每个十六进制“数字”.
```rust
fn execute(&mut self, op: u16) {
    let digit1 = (op & 0xF000) >> 12;
    let digit2 = (op & 0x0F00) >> 8;
    let digit3 = (op & 0x00F0) >> 4;
    let digit4 = op & 0x000F;
}
```

Perhaps not the cleanest code, but we need each hex digit separately. From here, we can create a `match` statement where we can specify the patterns for all of our opcodes:

可能不是最清晰的代码，但是，我们需要每个十六进制划分。从这儿，我们能够创建一个`match`语句在我们为所有的操作码的专门模式。

```rust
fn execute(&mut self, op: u16) {
    let digit1 = (op & 0xF000) >> 12;
    let digit2 = (op & 0x0F00) >> 8;
    let digit3 = (op & 0x00F0) >> 4;
    let digit4 = op & 0x000F;

    match (digit1, digit2, digit3, digit4) {
        (_, _, _, _) => unimplemented!("Unimplemented opcode: {}", op),
    }
}
```

Rust's `match` statement demands that all possible options be taken into account which is done with the `_` variable, which captures "everything else". Inside, we'll use the `unimplemented!` macro to cause the program to panic if it reaches that point. By the time we finish adding all opcodes, the Rust compiler demands that we still have an "everything else" statement, but we should never hit it.

Rust的`match`语句要求所有可能的操作用`_`的变量完成这些说明，其功能"everything else".在里面，当它到达了这个点，我们将使用`unimplemented`宏去导致程序陷入。同时，我们完成添加所有操作码,这个rust编译器要求是，我们仍然有一个"everything else"语句，但是我们将不会触发它。

While a long `match` statement would certainly work for other architectures, it is usually more common to implement instructions in their own functions, and either use a lookup table or programmatically determine which function is correct. Chip-8 is somewhat unusual because it stores instruction parameters into the opcode itself, meaning we need a lot of wild cards to match the instructions. Since there are a relatively small number of them, a `match` statement works well here.

当一个长的`match`语句将要确定作为其他架构的工作，它通常更一般地实现在它们拥有的函数里面的指令，也使用一个查找表或者可编程的确定哪个函数是正确的。chip-8和一般的有点不同，因为它存储指令参数在操作码本身，意味着我们需要许多卡片去匹配这些指令。因为有一些相关小的数字在他们里面，一个`match`语句在这儿可以很好工作。

With the framework setup, let's dive in!
利用框架的支持，让我们前进!

## Intro to Implementing Opcodes
## 指令码实现的介绍

The following pages individually discuss how all of Chip-8's instructions work, and include code of how to implement them. You are welcome to simply follow along and implement instruction by instruction, but before you do that, you may want to look forward to the [next section](#dfe) and begin working on some of the frontend code. Currently we have no way of actually running our emulator, and it may be useful to some to be able to attempt to load and run a game for debugging. However, do remember that the emulator will likely crash rather quickly unless all of the instructions are implemented. Personally, I prefer to work on the instructions first before working on the other moving parts (hence why this guide is laid out the way it is).

下面一些页面被分成讨论所有的chip-8'的指令如何工作，和包含怎样去实现它们。你可以简单沿着这个，
一个指令一个指令地来实现，但是在你做之前，你可能想要查找[下一段](#dfe),然后开始于前端上的一些代码的
工作。目前，我们没有精确的方式来运行我们的模拟器，这些可能是是有用的，能够尝试加载和为调试来
运行一个游戏。但是，要记住，模拟器将会很快崩溃，除非所有的指令已经实现。从个人角度来说，
在转移到其他部分之前，我宁可首先实现在指令上面（因此，这个向导是这样的方式来安排）。


With that disclaimer out of the way, let's proceed to working on each of the Chip-8 instructions in turn.

在排除方式后，让我们转到每个chip-8指令的处理工作上。

### 0000 - Nop

Our first instruction is the simplest one - do nothing. This may seem like a silly one to have, but sometimes it's needed for timing or alignment purposes. In any case, we simply need to move on to the next opcode (which was already done in `fetch`), and return.

我们的第一个指令是最简单的一个-什么都不做。这个可能看起来像一个愚蠢的，但是有时它们是作为
定时或者对齐需要。在一些情况，我们需要简单的移动到下一个操作码(其已经在`fetch`里完成),然后返回.

```rust
match (digit1, digit2, digit3, digit4) {
    // NOP
    (0, 0, 0, 0) => return,
    (_, _, _, _) => unimplemented!("Unimplemented opcode: {}", op),
}
```

### 00E0 - Clear screen
### 00E0 - 清屏
Opcode 0x00E0 is the instruction to clear the screen, which means we need to reset our screen buffer to be empty again.
操作码0x00E0是清屏指令,其意思是我们需要再次重设我们的屏幕的buffer到空。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // CLS
    (0, 0, 0xE, 0) => {
        self.screen = [false; SCREEN_WIDTH * SCREEN_HEIGHT];
    },

    // -- Unchanged code omitted --
}
```

### 00EE - Return from Subroutine
### 00EE - 从实例返回
We haven't yet spoken about subroutines (aka functions) and how they work. Entering into a subroutine works in the same way as a plain jump; we move the PC to the specified address and resume execution from there. Unlike a jump, a subroutine is expected to complete at some point, and we will need to return back to the point where we entered. This is where our stack comes in. When we enter a subroutine, we simply push our address onto the stack, run the routine's code, and when we're ready to return we pop that value off our stack and execute from that point again. A stack also allows us to maintain return addresses for nested subroutines while ensuring they are returned in the correct order.

我们还没有讲关于实例和他们怎样工作.进入一个实例的工作和作为一个普通的跳转是相同的方式;我们
转移PC到专门地址，然后假设从那儿开始执行。不想跳转，一个实例期待在一些点完成，然后，我们
需要返回到我们进入的地方。这就是我们栈进来的地方。当我们进入一个实例，我们简单弹入我们的
地址到栈，然后，运行实例代码，当我们需要返回值，我们弹出我们栈的值，然后那么调用的
那个点继续执行。一个栈允许我们保留必要实例的返回地址，以便它们按照正确顺序返回。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // RET
    (0, 0, 0xE, 0xE) => {
        let ret_addr = self.pop();
        self.pc = ret_addr;
    },

    // -- Unchanged code omitted --
}
```

### 1NNN - Jump
### 1NNN - 跳转
The jump instruction is pretty easy to add, simply move the PC to the given address.
一个跳转指令是相当容易被添加，简单移动PC到给定地址.
```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // JMP NNN
    (1, _, _, _) => {
        let nnn = op & 0xFFF;
        self.pc = nnn;
    },

    // -- Unchanged code omitted --
}
```

The main thing to notice here is that this opcode is defined by '0x1' being the most significant digit. The other digits are used as parameters for this operation, hence the `_` placeholder in our match statement, here we want anything starting with a 1, but ending in any three digits to enter this statement.

这里主要注意的点是，这个操作码被用'0x1'定义作为大多数的有符号的数字。其他数字被作为这个操作
的参数，因此`_`匹配我们语句的地方，这里，我们想要一些以一个1开始，但是以三个数字结束的来进入
这个语句。

### 2NNN - Call Subroutine
### 2NNN - 访问实例

The opposite of our 'Return from Subroutine' operation, we are going to add our current PC to the stack, and then jump to the given address. If you skipped straight here, I recommend reading the *Return* section for additional context.

和我们的'从实例返回'操作相反，我们将增加我们当前PC到栈，然后，跳转到给定地址。如果，你直接
跳过这里。我推荐阅读*Return*段来增加它的上下文。


```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // CALL NNN
    (2, _, _, _) => {
        let nnn = op & 0xFFF;
        self.push(self.pc);
        self.pc = nnn;
    },

    // -- Unchanged code omitted --
}
```

### 3XNN - Skip next if VX == NN
### 3XNN - 如果VX == NN 跳过下一个

This opcode is first of a few that follow a similar pattern. For those who are unfamiliar with assembly, being able to skip a line gives similar functionality to an if-else block. We can make a comparison, and if true go to one instruction, and if false go somewhere else. This is also the first opcode which will use one of our *V registers*. In this case, the second digit tells us which register to use, while the last two digits provide the raw value.

这个操作是第一个，后面跟着一些模式。对于那些还不熟悉汇编，类似于一个if-else块的能够跳过
一给定的行，这也是我们第一个使用*V 寄存器*的操作码。在这个情况下，当最后两个数字原始值时，
第二个数字告诉我们使用哪个寄存器，


```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // SKIP VX == NN
    (3, _, _, _) => {
        let x = digit2 as usize;
        let nn = (op & 0xFF) as u8;
        if self.v_reg[x] == nn {
            self.pc += 2;
        }
    },

    // -- Unchanged code omitted --
}
```

The implementation works like this: since we already have the second digit saved to a variable, we will reuse it for our 'X' index, although cast to a `usize`, as Rust requires all array indexing to be done with a `usize` variable. If that value stored in that register equals `nn`, then we skip the next opcode, which is the same as skipping our PC ahead by two bytes.
这部分实现像这样:因为我们已经有第二个数字被保存在一个变量里，我们将复用它作为我们的'X'索引,
虽然转换为一个`usize`，作为Rust需要所有的数组索引使用一个`usize`类型变量来完成。如果存储在那个
寄存器里面的值等于`nn`,然后，我们跳过下一个操作码，它和跳过我们PC向前两个字节是相同的。


### 4XNN - Skip next if VX != NN
### 4XNN - 如果VX不等于NN，跳过下一个

This opcode is exactly the same as the previous, except we skip if the compared values are not equal.

这个操作码和前面一个是尤其相同，除了如果比较的两个值不相等则我们跳过。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // SKIP VX != NN
    (4, _, _, _) => {
        let x = digit2 as usize;
        let nn = (op & 0xFF) as u8;
        if self.v_reg[x] != nn {
            self.pc += 2;
        }
    },

    // -- Unchanged code omitted --
}
```

### 5XY0 - Skip next if VX == VY
### 5XY0 - 如果VX等于VY 我们跳过下一个

A similar operation again, however we now use the third digit to index into another *V Register*. You will also notice that the least significant digit is not used in the operation. This opcode requires it to be 0.

再次一个类似的操作,但是，我们现在使用第三个数字作为索引作为另外*V 寄存器*.你将注意到，这最后符号
的数字没有用在我们的操作里。这个操作码需要它是0.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // SKIP VX == VY
    (5, _, _, 0) => {
        let x = digit2 as usize;
        let y = digit3 as usize;
        if self.v_reg[x] == self.v_reg[y] {
            self.pc += 2;
        }
    },

    // -- Unchanged code omitted --
}
```

### 6XNN - VX = NN

Set the *V Register* specified by the second digit to the value given.

通过第二个数字给出的值设置*V 寄存器*.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX = NN
    (6, _, _, _) => {
        let x = digit2 as usize;
        let nn = (op & 0xFF) as u8;
        self.v_reg[x] = nn;
    },

    // -- Unchanged code omitted --
}
```

### 7XNN - VX += NN

This operation adds the given value to the VX register. In the event of an overflow, Rust will panic, so we need to use a different method than the typical addition operator. Note also that while Chip-8 has a carry flag (more on that later), it is not modified by this operation.

这个操作增加给出的值到VX寄存器。在发生溢出时，Rust将会触发，所以，我们需要使用一个不同相比于传统的
加法操作方法。也需要注意的是，当chip-8有一个进位标记(后面可能更多)，这个操作没有发生修改.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX += NN
    (7, _, _, _) => {
        let x = digit2 as usize;
        let nn = (op & 0xFF) as u8;
        self.v_reg[x] = self.v_reg[x].wrapping_add(nn);
    },

    // -- Unchanged code omitted --
}
```

### 8XY0 - VX = VY

Like the `VX = NN` operation, but the source value is from the VY register.
类似`VX = NN`操作,但是值的来源于VY寄存器.
```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX = VY
    (8, _, _, 0) => {
        let x = digit2 as usize;
        let y = digit3 as usize;
        self.v_reg[x] = self.v_reg[y];
    },

    // -- Unchanged code omitted --
}
```

### 8XY1, 8XY2, 8XY3 - Bitwise operations
### 8XY1, 8XY2, 8XY3 - 位宽操作

The `8XY1`, `8XY2`, and `8XY3` opcodes are all similar functions, so rather than repeat myself three times over, I'll implement the *OR* operation, and allow the reader to implement the other two.

这`8XY1`, `8XY2`, 和 `8XY3` 操作码是类似的功能，所以，想大于自己三次。我将实现*OR*操作,然后允许读者去实现其他的两个.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX |= VY
    (8, _, _, 1) => {
        let x = digit2 as usize;
        let y = digit3 as usize;
        self.v_reg[x] |= self.v_reg[y];
    },

    // -- Unchanged code omitted --
}
```

### 8XY4 - VX += VY

This operation has two aspects to make note of. Firstly, this operation has the potential to overflow, which will cause a panic in Rust if not handled correctly. Secondly, this operation is the first to utilize the `VF` flag register. I've touched upon it previously, but while the first 15 *V* registers are general usage, the final 16th (0xF) register doubles as the *flag register*. Flag registers are common in many CPU processors; in the case of Chip-8 it also stores the *carry flag*, basically a special variable that notes if the last application operation resulted in an overflow/underflow. Here, if an overflow were to happen, we need to set the `VF` to be 1, or 0 if not. With these two aspects in mind, we will use Rust's `overflowing_add` attribute, which will return a tuple of both the wrapped sum, as well as a boolean of whether an overflow occurred.

这个操作有两个方面需要注意。首先，这个操作有潜在溢出，如果没有正确处理其将导致rust的
一个panic。第二，这个操作的第一个是使用`VF`标记寄存器。我们在前面已经接触过它，但是，
当前面的15个*V*寄存器作为一般使用，这个最后第16(0xF)寄存器疑似作为*标记寄存器*. 在一些
CPU处理器中是一些普通的标记寄存器;在这个chip-8的情况，它也存储*进位标记*,基本上一个专有变量，
去记录是否最后的应用程序操作产生上溢/下溢.这里，如果一个上溢发生，我们需要设置`VF`为1,
或者如果没有，我们设置为0.在我们头脑中保留这两方面,我们将使用Rust的`overflowing_add`属性,
其将返回包含结果的一个元组,正如是否一个溢出发生的布尔值.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX += VY
    (8, _, _, 4) => {
        let x = digit2 as usize;
        let y = digit3 as usize;

        let (new_vx, carry) = self.v_reg[x].overflowing_add(self.v_reg[y]);
        let new_vf = if carry { 1 } else { 0 };

        self.v_reg[x] = new_vx;
        self.v_reg[0xF] = new_vf;
    },

    // -- Unchanged code omitted --
}
```
### 8XY5 - VX -= VY

This is the same operation as the previous opcode, but with subtraction rather than addition. The key distinction is that the `VF` carry flag works in the opposite fashion. The addition operation would set the flag to 1 if an overflow occurred, here if an underflow occurs, it is set to 0, and vice versa. The `overflowing_sub` method will be of use to us here.

这个和前面操作码是相同的操作，但是相比于加法，这里是减法.关键的不同点是在于'VF'进位标记是
相反产生。如果溢出发生加法操作将设置标记为1，这里是如果一个借位发生，它将设置为0，
反之亦然。这个`overflowing_sub`方法将在这里使用。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX -= VY
    (8, _, _, 5) => {
        let x = digit2 as usize;
        let y = digit3 as usize;

        let (new_vx, borrow) = self.v_reg[x].overflowing_sub(self.v_reg[y]);
        let new_vf = if borrow { 0 } else { 1 };

        self.v_reg[x] = new_vx;
        self.v_reg[0xF] = new_vf;
    },

    // -- Unchanged code omitted --
}
```

### 8XY6 - VX >>= 1

This operation performs a single right shift on the value in VX, with the bit that was dropped off being stored into the `VF` register. Unfortunately, there isn't a built-in Rust `u8` operator to catch the dropped bit, so we will have to do it ourself.

这个操作执行一个简单的在VX里面值右移，把被移出的位保存到`VF`寄存器里。不幸的是，Rust这里没有
构建`u8`操作来捕获移出位，所以，我们将自己来完成它.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX >>= 1
    (8, _, _, 6) => {
        let x = digit2 as usize;
        let lsb = self.v_reg[x] & 1;
        self.v_reg[x] >>= 1;
        self.v_reg[0xF] = lsb;
    },

    // -- Unchanged code omitted --
}
```

### 8XY7 - VX = VY - VX

This operation works the same as the previous VX -= VY operation, but with the operands in the opposite direction.

这个指令工作和前面的VX -= VY操作是相同的，但是使用相反方向的操作数.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX = VY - VX
    (8, _, _, 7) => {
        let x = digit2 as usize;
        let y = digit3 as usize;

        let (new_vx, borrow) = self.v_reg[y].overflowing_sub(self.v_reg[x]);
        let new_vf = if borrow { 0 } else { 1 };

        self.v_reg[x] = new_vx;
        self.v_reg[0xF] = new_vf;
    },

    // -- Unchanged code omitted --
}
```

### 8XYE - VX <<= 1

Similar to the right shift operation, but we store the value that is overflowed in the flag register.

类似于右移操作，但是我们把溢出的值在标记寄存器里。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX <<= 1
    (8, _, _, 0xE) => {
        let x = digit2 as usize;
        let msb = (self.v_reg[x] >> 7) & 1;
        self.v_reg[x] <<= 1;
        self.v_reg[0xF] = msb;
    },

    // -- Unchanged code omitted --
}
```

### 9XY0 - Skip if VX != VY
### 9XY0 - 如果VX != VY 跳过
Done with the 0x8000 operations, it's time to go back and add an opcode that was notably missing, skipping the next line if VX != VY. This is the same code as the 5XY0 operation, but with an inequality.

在完成0x8000操作之后，它再次回来，增加一个操作码是很少见的，如果VX != VY跳过下一行。这个
和5XY0操作是相同的，但是使用不相等.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // SKIP VX != VY
    (9, _, _, 0) => {
        let x = digit2 as usize;
        let y = digit3 as usize;
        if self.v_reg[x] != self.v_reg[y] {
            self.pc += 2;
        }
    },

    // -- Unchanged code omitted --
}
```

### ANNN - I = NNN

This is the first instruction to utilize the *I Register*, which will be used in several additional instructions, primarily as an address pointer to RAM. In this case, we are simply setting it to the 0xNNN value encoded in this opcode.

这是第一个使用*I寄存器*的指令，它将被用于一些添加的指令，基本是作为一个地址指向内存.在这个
情况，我们将简单设置它到0xNNN的值在这个操作码里.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // I = NNN
    (0xA, _, _, _) => {
        let nnn = op & 0xFFF;
        self.i_reg = nnn;
    },

    // -- Unchanged code omitted --
}
```

### BNNN - Jump to V0 + NNN

While previous instructions have used the *V Register* specified within the opcode, this instruction always uses the first *V0* register. This operation moves the PC to the sum of the value stored in *V0* and the raw value 0xNNN supplied in the opcode.

当前面指令已经使用*V寄存器*表示在操作码里，这个指令总是使用第一个*V0*寄存器。这个指令移动PC到
存贮在*V0*里面的值与操作里面提供的原始0xNNN的值的和。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // JMP V0 + NNN
    (0xB, _, _, _) => {
        let nnn = op & 0xFFF;
        self.pc = (self.v_reg[0] as u16) + nnn;
    },

    // -- Unchanged code omitted --
}
```

### CXNN - VX = rand() & NN

Finally, something to shake up the monotony! This opcode is Chip-8's random number generation, with a slight twist, in that the random number is then AND'd with the lower 8-bits of the opcode. While the Rust development team has released a random generation crate, it is not part of its standard library, so we shall have to add it to our project.

最后，一些事情将打破无聊！这个操作码是Chip-8的随机数生成，使用一个轻微的变化是在这个
随机数里，然后，使用操作码的低8位进行AND.当rust开发团队已经发布了一个随机生成的包，它不是
标准库的一部分，所以，我们将必须增加它到我们项目。

In `chip8_core/Cargo.toml`, add the following line somewhere under `[dependencies]`:

```toml
rand = "^0.7.3"
```

Note: If you are planning on following this guide completely to its end, there will be a future change to how we include this library for web browser support. However, at this stage in the project, it is enough to specify it as is.

Time now to add RNG support and implement this opcode. At the top of `lib.rs`, we will need to import a function from the `rand` crate:
是时候增加RNG支持和实现这个操作码。在`lib.rs`,我们将需要从`rand`包导入一个函数:

```rust
use rand::random;
```

We will then use the `random` function when implementing our opcode:
然后，我们将使用`random`函数实现我们的操作码:
```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX = rand() & NN
    (0xC, _, _, _) => {
        let x = digit2 as usize;
        let nn = (op & 0xFF) as u8;
        let rng: u8 = random();
        self.v_reg[x] = rng & nn;
    },

    // -- Unchanged code omitted --
}
```

Note that specifying `rng` as a `u8` variable is necessary for the `random()` function to know which type it is supposed to generate.

注意那个声明`rng`是作为一个`u8`变量，对于`random()`函数有必要知道哪种支持类型被生成.

### DXYN - Draw Sprite
### DXYN - 绘制精灵

This is probably the single most complicated opcode, so allow me to take a moment to describe how it works in detail. Rather than drawing individual pixels or rectangles to the screen at a time, the Chip-8 display works by drawing *sprites*, images stored in memory that are copied to the screen at a specified (x, y). For this opcode, the second and third digits give us which *V Registers* we are to fetch our (x, y) coordinates from. So far so good. Chip-8's sprites are always 8 pixels wide, but can be a variable number of pixels tall, from 1 to 16. This is specified in the final digit of our opcode. I mentioned earlier that the *I Register* is used frequently to store an address in memory, and this is the case here; our sprites are stored row by row *beginning* at the address stored in *I*. So if we are told to draw a 3px tall sprite, the first row's data is stored at \*I, followed by \*I + 1, then \*I + 2. This explains why all sprites are 8 pixels wide, each row is assigned a byte, which is 8-bits, one for each pixel, black or white. The last detail to note is that if *any* pixel is flipped from white to black or vice versa, the *VF* is set (and cleared otherwise). With these things in mind, let's begin.

这个可能是一个最复杂的操作码，所以，允许我们花一点时间来描述它是怎样工作的详情。相当于一次绘制
独立像素或者矩形到屏幕，chip-8使用绘制*精灵*进行显示工作，存储在内存里面的图像被考妣到屏幕上一个
指定(x,y)位置处. 对于这个操作码，第二和第三存放在*V 寄存器*里的数字提供给我们想要从哪儿遍历的
(x,y)坐标,到目前为止都很好。chip-8的精灵总是8个像素宽，但是，可以是一个不同像素高，从1到16.
这个被表示在操作码的最后一个数字里。我早期提及过*I寄存器*被频繁地用于存储地址在内存里，这里
就是案例;我们的精灵被以一行接一行的方式开始于在*I*里的位置进行存储。所以，如果我们被告知绘制
一个3像素高的精灵，这第一行的数据被存放在\*I，接着是\*I +1, 然后\*I +2. 这个解释了为什么精灵是
8像素宽，每行被赋予一个字节，其是8位，一个像素一位,黑色或者白色。这最后需要注意的是，如果
*一些*像素是从白到黑翻转或者相反，这个*VF*被设置(其他情况清除).在我们大脑里使用这些，让我们开始.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // DRAW
    (0xD, _, _, _) => {
        // Get the (x, y) coords for our sprite
        let x_coord = self.v_reg[digit2 as usize] as u16;
        let y_coord = self.v_reg[digit3 as usize] as u16;
        // The last digit determines how many rows high our sprite is
        let num_rows = digit4;

        // Keep track if any pixels were flipped
        let mut flipped = false;
        // Iterate over each row of our sprite
        for y_line in 0..num_rows {
            // Determine which memory address our row's data is stored
            let addr = self.i_reg + y_line as u16;
            let pixels = self.ram[addr as usize];
            // Iterate over each column in our row
            for x_line in 0..8 {
                // Use a mask to fetch current pixel's bit. Only flip if a 1
                if (pixels & (0b1000_0000 >> x_line)) != 0 {
                    // Sprites should wrap around screen, so apply modulo
                    let x = (x_coord + x_line) as usize % SCREEN_WIDTH;
                    let y = (y_coord + y_line) as usize % SCREEN_HEIGHT;

                    // Get our pixel's index for our 1D screen array
                    let idx = x + SCREEN_WIDTH * y;
                    // Check if we're about to flip the pixel and set
                    flipped |= self.screen[idx];
                    self.screen[idx] ^= true;
                }
            }
        }

        // Populate VF register
        if flipped {
            self.v_reg[0xF] = 1;
        } else {
            self.v_reg[0xF] = 0;
        }
    },

    // -- Unchanged code omitted --
}
```

### EX9E - Skip if Key Pressed
### EX9E - 如果有键按下就跳过

Time at last to introduce user input. When setting up our emulator object, I mentioned that there are 16 possible keys numbered 0 to 0xF. This instruction checks if the index stored in VX is pressed, and if so, skips the next instruction.

是时候介绍用户输入了。当设置我们模拟器对象，我提及过，这里有16个可能得键值，从0到0xF。这个指令检查
是否存储在VX里的索引被按下，如果是，跳过下一个指令。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // SKIP KEY PRESS
    (0xE, _, 9, 0xE) => {
        let x = digit2 as usize;
        let vx = self.v_reg[x];
        let key = self.keys[vx as usize];
        if key {
            self.pc += 2;
        }
    },

    // -- Unchanged code omitted --
}
```

### EXA1 - Skip if Key Not Pressed
### EXA1 - 如果没有按键就跳过

Same as the previous instruction, however this time the next instruction is skipped if the key in question is not being pressed.
和前面指令相同，但是，这次是如果问题里的按照没有被按下后，跳过下一个指令。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // SKIP KEY RELEASE
    (0xE, _, 0xA, 1) => {
        let x = digit2 as usize;
        let vx = self.v_reg[x];
        let key = self.keys[vx as usize];
        if !key {
            self.pc += 2;
        }
    },

    // -- Unchanged code omitted --
}
```

### FX07 - VX = DT

I mentioned the use of the *Delay Timer* when we were setting up the emulation structure. This timer ticks down every frame until reaching zero. However, that operation happens automatically, it would be really useful to be able to actually see what's in the *Delay Timer* for our game's timing purposes. This instruction does just that, and stores the current value into one of the *V Registers* for us to use.

当我们设置模拟器结构时，我提及过使用*延迟时钟*.这个时钟，每帧滴答下降1直到到达0.但是，这个操作是自动
发生的，它将是真真有用于能够实际看到我们用于游戏计时在*延时时钟*里。这个指令做这个，然后，存储
当前的值到*V 寄存器*中的一个给我们使用.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // VX = DT
    (0xF, _, 0, 7) => {
        let x = digit2 as usize;
        self.v_reg[x] = self.dt;
    },

    // -- Unchanged code omitted --
}
```

### FX0A - Wait for Key Press
### FX0A - 等待按键

While we already had instructions to check if keys are either pressed or released, this instruction does something very different. Unlike those, which checked the key state and then moved on, this instruction is *blocking*, meaning the whole game will pause and wait for as long as it needs to until the player presses a key. That means it needs to loop endlessly until something in our `keys` array turns true. Once a key is found, it is stored into VX. If more than one key is currently being pressed, it takes the lowest indexed one.

当我们已经有指令检查键是按下还是释放，这个指令所做的是非常不同的。不像哪些检查键状态，然后
移动。这个指令是*阻塞*,意味着整个游戏会暂停，等待足够长直到玩家按一个键。这意味着它需要无限制
循环知道我们的一些*键*变成true。一旦一个键被发现，它存储到VX里。如果当前多于一个键被按下，它
将使用最低的索引.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // WAIT KEY
    (0xF, _, 0, 0xA) => {
        let x = digit2 as usize;
        let mut pressed = false;
        for i in 0..self.keys.len() {
            if self.keys[i] {
                self.v_reg[x] = i as u8;
                pressed = true;
                break;
            }
        }

        if !pressed {
            // Redo opcode
            self.pc -= 2;
        }
    },

    // -- Unchanged code omitted --
}
```

You may be looking at this implementation and asking "why are we resetting the opcode and going through the entire fetch sequence again, rather than simply doing this in a loop?". Simply put, while we want this instruction to block future instructions from running, we do not want to block any new key presses from being registered. By remaining in a loop, we would prevent our key press code from ever running, causing this loop to never end. Perhaps inefficient, but much simpler than some sort of asynchronous checking.

你可能寻找实现和问"为什么我们重设操作码，然后通过入口再次遍历序列，相比于简单地在一个
循环里面操作?".简单说，当我们想要这个指令从当前运行状态去阻塞后面指令，我们不像要从已经注册
过的阻塞一些新的键按下。在一个循环剩下部分，我们将杜绝我们来自曾经运行的按键码，导致这个循环
无法结束，可能没有影响，但是相比一些同步检查更简单。

### FX15 - DT = VX

This operation works the other direction from our previous *Delay Timer* instruction. We need someway to reset the *Delay Timer* to a value, and this instruction allows us to copy over a value from a *V Register* of our choosing.

这个操作码工作在来自我们前面*延迟时间*另外一个方向。我们需要一些方式重设*延迟时钟*到一个值，
这个指令允许我们拷贝一个来自我们选择的*V 寄存器*的值。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // DT = VX
    (0xF, _, 1, 5) => {
        let x = digit2 as usize;
        self.dt = self.v_reg[x];
    },

    // -- Unchanged code omitted --
}
```

### FX18 - ST = VX

Almost the exact same instruction as the previous, however this time we are going to store the value from VX into our *Sound Timer*.

大部分执行和前面相同的指令，不过，这次我们将存贮来自VX的值到我们的*声音时钟*.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // ST = VX
    (0xF, _, 1, 8) => {
        let x = digit2 as usize;
        self.st = self.v_reg[x];
    },

    // -- Unchanged code omitted --
}
```

### FX1E - I += VX

Instruction ANNN sets I to the encoded 0xNNN value, but sometimes it is useful to be able to simply increment the value. This instruction takes the value stored in VX and adds it to the *I Register*. In the case of an overflow, the register should simply roll over back to 0, which we can accomplish with Rust's `wrapping_add` method.

指令ANNN设置I到编码成0xNNN的值，但是，有时它是有用的能够简单增加值。这个指令使用保存在VX
里的值，然后增加它到*I寄存器*。在溢出情况下，寄存器将简单回滚到0，我们能够使用Rust
的`wrapping_add`方法来完成。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // I += VX
    (0xF, _, 1, 0xE) => {
        let x = digit2 as usize;
        let vx = self.v_reg[x] as u16;
        self.i_reg = self.i_reg.wrapping_add(vx);
    },

    // -- Unchanged code omitted --
}
```

### FX29 - Set I to Font Address
### FX29  - 设置I到字体地址

This is another tricky instruction where it may not be clear how to progress at first. If you recall, we stored an array of font data at the very beginning of RAM when initializing the emulator. This instruction wants us to take from VX a number to print on screen (from 0 to 0xF), and store the RAM address of that sprite into the *I Register*. We are actually free to store those sprites anywhere we wanted, so long as we are consistent and point to them correctly here. However, we stored them in a very convenient location, at the beginning of RAM. Let me show you what I mean by printing out some of the sprites and their RAM locations.

这是另外一个巧妙的指令，一开始不能够清楚知道怎样处理。如果你回想下，当初始化模拟器时候，我们
存储字体数组数据在内存的比较开始地方。这个指令想要我们从VX使用一个数字去打印在屏幕上(从0
到0xF)，然后，存储精灵的内存地址到*I寄存器*。我们可以相当自由存储这些精灵到我们想要的任何
地方，正如我们考虑的，在内存初始位置.让我来给你展示，我的意思是通过答应一些精灵和他们的内存
位置.

| Character | RAM Address |
| --------- | ----------- |
| 0         | 0           |
| 1         | 5           |
| 2         | 10          |
| 3         | 15          |
| ...       | ...         |
| E (14)    | 70          |
| F (15)    | 75          |

You'll notice that since all of our font sprites take up five bytes each, their RAM address is simply their value times 5. If we happened to store the fonts in a different RAM address, we could still follow this rule, however we'd have to apply an offset to where the block begins.

你已经注意到，因为我们所有的字体精灵使用每个5个字节,它们的内存地址是简单的它们值的5倍。如果
我们发生存储字体在一个不同的内存地址，我们可能不能遵循这个规则，但是，我们必须应用一个偏移
到块的开始处。

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // I = FONT
    (0xF, _, 2, 9) => {
        let x = digit2 as usize;
        let c = self.v_reg[x] as u16;
        self.i_reg = c * 5;
    },

    // -- Unchanged code omitted --
}
```

### FX33 - I = BCD of VX

Most of the instructions for Chip-8 are rather self-explanitory, and can be implemented quite easily just by hearing a vague description. However, there are a few that are quite tricky, such as drawing to a screen and this one, storing the [Binary-Coded Decimal](https://en.wikipedia.org/wiki/Binary-coded_decimal) of a number stored in VX into the *I Register*. I encourage you to read up on BCD if you are unfamiliar with it, but a brief refresher goes like this. In this tutorial, we've been using hexadecimal quite a bit, which works by converting our normal decimal numbers into base-16, which is more easily understood by computers. For example, the decimal number 100 would become 0x64. This is good for computers, but not very accessible to humans, and certainly not to the general audience who are going to play your games. The main purpose of BCD is to convert a hexadecimal number back into a pseudo-decimal number to print out for the user, such as for your points or high scores. So while Chip-8 might store 0x64 internally, fetching its BCD would give us `0x1, 0x0, 0x0`, which we could print to the screen as "100". You'll notice that we've gone from one byte to three in order to store all three digits of our number, which is why we are going to store the BCD into RAM, beginning at the address currently in the *I Register* and moving along. Given that VX stores 8-bit numbers, which range from 0 to 255, we are always going to end up with three bytes, even if some are zero.

对于chip-8大多数指令是自解释的，我们能够相当容易实现，仅通过听一个大概描述。但是，这里有一些
是相当诡异，比如绘制到屏幕上，和这个。存储一个保存在VX里数字的[二进制编码的十进制数](https://en.wikipedia.org/wiki/Binary-coded_decimal)到*I寄存器*。如果你不熟悉，我鼓励你去阅读BCD,但是简单引用像这样。在这个手册，我们已经使用一个16进制作为一个位,使用转换我们一般10进制数到基于16进制
的，它能够更容易被计算机所理解。例如，10进制数100将变成0x64.这个对于计算机很友好，但是对于
人类不是很好接受，不是确信一般的观众将玩你的游戏。这个BCD的目的是转换一个16进制数字回一个
伪10进制数给用户打印出。比如你的点数或者高分。所以，当chip-8可能内部存储0x64,遍历它们的BCD
将给我们`0x1,0x0,0x0`,我们将在屏幕上打印如"100".你将注意到，我们将从一个字节到三个为了存储我们
数的三个数字，这个就是为什么我将要存储BCD到内存，开始于当前*I寄存器*位置，然后沿着移动。给出
VX存储8位数字，它的范围从0到255，我们总是将要使用3个字节终止，即使一些是0.

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // BCD
    (0xF, _, 3, 3) => {
        let x = digit2 as usize;
        let vx = self.v_reg[x] as f32;

        // Fetch the hundreds digit by dividing by 100 and tossing the decimal
        let hundreds = (vx / 100.0).floor() as u8;
        // Fetch the tens digit by dividing by 10, tossing the ones digit and the decimal
        let tens = ((vx / 10.0) % 10.0).floor() as u8;
        // Fetch the ones digit by tossing the hundreds and the tens
        let ones = (vx % 10.0) as u8;

        self.ram[self.i_reg as usize] = hundreds;
        self.ram[(self.i_reg + 1) as usize] = tens;
        self.ram[(self.i_reg + 2) as usize] = ones;
    },

    // -- Unchanged code omitted --
}
```

For this implementation, I converted our VX value first into a `float`, so that I could use division and modulo arithmetic to get each decimal digit. This is not the fastest implementation nor is it probably the shortest. However, it is one of the easiest to understand. I'm sure there are some highly binary-savvy readers who are disgusted that I did it this way, but this solution is not for them. This is for readers who have never seen BCD before, where losing some speed for greater understanding is a better trade-off. However, once you have this implemented, I would encourage everyone to go out and look up more efficient BCD algorithms to add a bit of easy optimization into your code.

对于这个实现，我们首先转换我们的VX值到一个`float`,所以，我们能够使用触发和取余算术运算来获得
每个10进制数字。这个不是最快的视线也不可能是最短的。但是，它是最容易理解的。我确信，这里有高水平
二进制读者能够鉴定我使用的这个方式。但是，但是这个解决方案不是为他们的。这是为那些不曾见识
过BCD的读者，为了更好理解失去一些速度是更好的交易。但是，一旦你有了这个实现，我们将鼓励
每个人跳出这里，寻找更有小的BCD算法来增加一些容易优化到你的代码。

### FX55 - Store V0 - VX into I

We're on the home stretch! These final two instructions populate our *V Registers* V0 thru the specified VX (inclusive) with the same range of values from RAM, beginning with the address in the *I Register*. This first one stores the values into RAM, while the next one will load them the opposite way.

我们在主页的扩展。这最后两个指令弹出我们的*V寄存器*,从V0到指定VX，从内存使用相同的范围值，
开始于在*I寄存器*地址。当下一个将加载它们以相反方式，这第一个存储到内存，

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // STORE V0 - VX
    (0xF, _, 5, 5) => {
        let x = digit2 as usize;
        let i = self.i_reg as usize;
        for idx in 0..=x {
            self.ram[i + idx] = self.v_reg[idx];
        }
    },

    // -- Unchanged code omitted --
}
```

### FX65 - Load I into V0 - VX

```rust
match (digit1, digit2, digit3, digit4) {
    // -- Unchanged code omitted --

    // LOAD V0 - VX
    (0xF, _, 6, 5) => {
        let x = digit2 as usize;
        let i = self.i_reg as usize;
        for idx in 0..=x {
            self.v_reg[idx] = self.ram[i + idx];
        }
    },

    // -- Unchanged code omitted --
}
```

### Final Thoughts

That's it! With this, we now have a fully implemented Chip-8 CPU. You may have noticed a lot of possible opcode values are never covered, particularly in the 0x0000, 0xE000, and 0xF000 ranges. This is okay. These opcodes are left as undefined by the original design, and thus if any game attempts to use them it will lead to a runtime panic. If you are still curious following the completion of this emulator, there are a number of Chip-8 extensions which do fill in some of these gaps to add additional functionality, but they will not be covered by this guide.

就这些。基于这个，我们现在已经能够完全实现chip-8cpu.你可能已经注意到，一些可能的操作码没有被
覆盖，实际上在0x000,0xE000,和0xF000范围。这个没关系的，这些操作码是原作者设计留下的，因此，
如果一些游戏尝试使用它们将会导致一个运行panic.如果你仍然好奇这个模拟器的完整，有一些chip-8
扩展，添加增加的功能来填充这里的一些间隙，但是它们将不在本手册覆盖.
\newpage
