# Implementing Emulation Methods
# 实现模拟器的方法
We have now created our `Emu` struct and defined a number of variables for it to manage, as well as defined an initialization function. Before we move on, there are a few useful methods we should add to our object now which will come in use once we begin implementation of the instructions.

我们现在创建我们的`Emu`结构和定义变量来管理它，像我们初始化函数所定义的。在我们转移之前，这里有一些有用的方法我们需要添加到我们对象中，这些在我们开始实现指令时将会再次被使用。

## Push and Pop
## 推入和弹出

We have added both a `stack` array, as well as a pointer `sp` to manage the CPU's stack, however it will be useful to implement both a `push` and `pop` method so we can access it easily.

我们已经添加了一个`stack`数组，同时一个指针`sp`来管理cpu的栈，但是，实现一个`push`和`pop`方法是有用的，以致我们能容易访问它。

```rust
impl Emu {
    // -- Unchanged code omitted --

    fn push(&mut self, val: u16) {
        self.stack[self.sp as usize] = val;
        self.sp += 1;
    }

    fn pop(&mut self) -> u16 {
        self.sp -= 1;
        self.stack[self.sp as usize]
    }

    // -- Unchanged code omitted --
}
```

These are pretty straightforward. `push` adds the given 16-bit value to the spot pointed to by the Stack Pointer, then moves the pointer to the next position. `pop` performs this operation in reverse, moving the SP back to the previous value then returning what is there. Note that attempting to pop an empty stack results in an underflow panic[^1]. You are welcome to add extra handling here if you like, but in the event this were to occur, that would indicate a bug with either our emulator or the game code, so I feel that a complete panic is acceptable.

这些相当完美。`push`添加给定的16位值到栈指针所指向的地方，然后移动指针到下一个位置。`pop`执行和上面相反的操作，移动SP回到前面的地方，然后返回这里的值。注意，尝试弹出一个空栈结果会陷入向下溢出。如果你喜欢，欢迎在这儿加入额外处理，但是即使这样的发生，会表示我们模拟器或者游戏代码的一个bug，所以，我感觉是一个可以接受的完美的疼痛。

## Font Sprites
## 字体精灵
We haven't yet delved into how the Chip-8 screen display works, but the gist for now is that it renders *sprites* which are stored in memory to the screen, one line at a time. It is up to the game developer to correctly load their sprites before copying them over. However wouldn't it be nice if the system automatically had sprites for commonly used things, such as numbers? I mentioned earlier that our PC will begin at address 0x200, leaving the first 512 intentionally empty. Most modern emulators will use that space to store the sprite data for font characters of all the hexadecimal digits, that is characters of 0-9 and A-F. We could store this data at any fixed position in RAM, but this space is already defined as empty anyway. Each character is made up of five rows of eight pixels, with each row using a byte of data, meaning that each letter altogether takes up five bytes of data. The following diagram illustrates how a character is stored as bytes.

我们还没有研究chip-8屏幕怎样显示的工作，但是现在的要点是它存储在内存里所渲染的*精灵*到屏幕，一次一条线。这个游戏开发者在拷贝它们之前正确地加载它们的精灵。但是，如果系统自动使用一般的东西，比如数字？我曾提及，我们的PC将在地址0x200处开始，留下一开始的512的单独空的。大多数现代模拟器将使用这个空间来存放十六进制格式的字体符号的精灵数据，这些事0-9和A-F的字符串。我们将存储这些数据在内存固定地方，但是，这个空间已经作为空的而定义。每个字符被制作成每行8个像素的5行，每行使用一个字节数据，意味着每个字符将使用5个字节。下面数字表示一个字符怎样作为字节存储的。

[^1] *Underflow* is when the value of an unsigned variable goes from above zero to below zero. In some languages the value would then "roll over" to the highest possible size, but in Rust this leads to a runtime error and needs to be handled differently if desired. The same goes for values exceeding the maximum possible value, known as *overflow*.

\newpage

![Chip-8 Font Sprite](img/font_diagram.png)

On the right, each row is encoded into binary. Each pixel is assigned a bit, which corresponds to whether that pixel will be white or black. *Every* sprite in Chip-8 is eight pixels wide, which means a pixel row requires 8-bits (1 byte). The above diagram shows the layout of the "1" character sprite. The sprites don't need all 8 bits of width, so they all have black right halves. Sprites have been created for all of the hexadecimal digits, and are required to be present somewhere in RAM for some games to function. Later in this guide we will cover the instruction that handles these sprites, which will show how these are loaded and how the emulator knows where to find them. For now, we simply need to define them. We will do so with a constant array of bytes; at the top of `lib.rs`, add:

在右边，每行被编码成二进制。每个像素赋予一位，其对应那个像素是白色或者黑色。chip-8里的`每个`精灵是8像素宽，意味着一个像素行需要8位(1个字节）.上面图片显示"1"的字符精灵的布局。这个精灵不需要全是8位宽，所以，它们有右边一半的黑色。精灵被使用十六进制数字所创建，对于一些游戏功能需要表示在内存的一些地方。在我们向导后面，我们将覆盖这些精灵的指令，其将显示这些怎样被加载和模拟器怎么知道在哪儿找到他们。现在，我们简单定义他们。我们将使用一个常量字节数组；在`lib.rs`的顶部，添加:

```rust
const FONTSET_SIZE: usize = 80;

const FONTSET: [u8; FONTSET_SIZE] = [
    0xF0, 0x90, 0x90, 0x90, 0xF0, // 0
    0x20, 0x60, 0x20, 0x20, 0x70, // 1
    0xF0, 0x10, 0xF0, 0x80, 0xF0, // 2
    0xF0, 0x10, 0xF0, 0x10, 0xF0, // 3
    0x90, 0x90, 0xF0, 0x10, 0x10, // 4
    0xF0, 0x80, 0xF0, 0x10, 0xF0, // 5
    0xF0, 0x80, 0xF0, 0x90, 0xF0, // 6
    0xF0, 0x10, 0x20, 0x40, 0x40, // 7
    0xF0, 0x90, 0xF0, 0x90, 0xF0, // 8
    0xF0, 0x90, 0xF0, 0x10, 0xF0, // 9
    0xF0, 0x90, 0xF0, 0x90, 0x90, // A
    0xE0, 0x90, 0xE0, 0x90, 0xE0, // B
    0xF0, 0x80, 0x80, 0x80, 0xF0, // C
    0xE0, 0x90, 0x90, 0x90, 0xE0, // D
    0xF0, 0x80, 0xF0, 0x80, 0xF0, // E
    0xF0, 0x80, 0xF0, 0x80, 0x80  // F
];
```

You can see the bytes outlined in the "1" diagram above, all of the other letters work in a similar way. Now that these are outlined, we need to load them into RAM. Modify `Emu::new()` to copy those values in:

你能看到上面图片中的"1"的字节的轮廓，所有其他的字母类似这样的工作。现在，有这些外形，我们需要加载他们到内存。修改`Emu::new()`里面，在里面拷贝这些值：

```rust
pub fn new() -> Self {
    let mut new_emu = Self {
        pc: START_ADDR,
        ram: [0; RAM_SIZE],
        screen: [false; SCREEN_WIDTH * SCREEN_HEIGHT],
        v_reg: [0; NUM_REGS],
        i_reg: 0,
        sp: 0,
        stack: [0; STACK_SIZE],
        keys: [false; NUM_KEYS],
        dt: 0,
        st: 0,
    };

    new_emu.ram[..FONTSET_SIZE].copy_from_slice(&FONTSET);

    new_emu
}
```

This initializes our `Emu` object in the same way as before, but copies in our character sprite data into RAM before returning it.

这里像之前一样初始化我们的`Emu`对象，但是在返回之前拷贝我们的字符精灵数据到内存.

It will also be useful to be able to reset our emulator without having to create a new object. There are fancier ways of doing this, but we'll just keep it simple and create a function that resets our member variables back to their original values when called.

不需要创建一个新对象能够重设我们模拟器是一种有效方式。有一些有趣的方式做这些，但是，我们将保持简单，创建一个函数重设我们的成员变量到它们原先被访问时候的值。

```rust
pub fn reset(&mut self) {
    self.pc = START_ADDR;
    self.ram = [0; RAM_SIZE];
    self.screen = [false; SCREEN_WIDTH * SCREEN_HEIGHT];
    self.v_reg = [0; NUM_REGS];
    self.i_reg = 0;
    self.sp = 0;
    self.stack = [0; STACK_SIZE];
    self.keys = [false; NUM_KEYS];
    self.dt = 0;
    self.st = 0;
    self.ram[..FONTSET_SIZE].copy_from_slice(&FONTSET);
}
```

## Tick
## 滴答
With the creation of our `Emu` object completed (for now), we can begin to define how the CPU will process each instruction and move through the game. To summarize what was described in the previous parts, the basic loop will be:

1. Fetch the value from our game (loaded into RAM) at the memory address stored in our Program Counter.
2. Decode this instruction.
3. Execute, which will possibly involve modifying our CPU registers or RAM.
4. Move the PC to the next instruction and repeat.

在我们的`Emu`对象完全创建，我们开始定义CPU将怎样处理每个指令和移动游戏。为了概括我们前面部分描述的内容，这个基本循环将是:
1. 遍历从我们程序计数器里面指向内存地址所存放的游戏数据值；
2. 解码这个指令；
3. 执行，它将尽可能必要修改我们CPU寄存器或者内存.
4. 移动PC到下一个指令，然后重复这个过程.

Let's begin by adding the opcode processing to our `tick` function, beginning with the fetching step:
让我们通过添加操作码处理到我们`tick`函数，开始遍历步骤:

```rust
// -- Unchanged code omitted --

pub fn tick(&mut self) {
    // Fetch
    let op = self.fetch();
    // Decode
    // Execute
}

fn fetch(&mut self) -> u16 {
    // TODO
}

```

The `fetch` function will only be called internally as part of our `tick` loop, so it doesn't need to be public. The purpose of this function is to grab the instruction we are about to execute (known as an *opcode*) for use in the next steps of this cycle. If you're unfamiliar with Chip-8's instruction format, I recommend you refresh up with the [overview](#eb) from the earlier chapters.

这个`fetch`函数将仅是作为我们内部`tick`循环的部分被访问,所以它不需要成为public。这个函数的目的是获取我们打算在本周期使用的下一个步骤的指令。如果你不熟悉Chip-8的指令格式，我推荐你从前面章节的[overview](#eb).

Fortunately, Chip-8 is easier than many systems. For one, there's only 35 opcodes to deal with as opposed to the hundreds that many processors support. In addition, many systems store additional parameters for each opcode in subsequent bytes (such as operands for addition), Chip-8 encodes these into the opcode itself. Due to this, all Chip-8 opcodes are exactly 2 bytes, which is larger than some other systems, but the entire instruction is stored in those two bytes, while other contemporary systems might consume between 1 and 3 bytes per cycle.

幸运的是,Chip-8相对其他系统是更容易。比如，相对于许多处理系统支持上百的操作码，这里仅有35个操作码的处理。因此，许多系统存储增加参数为在接下来字节的每个操作码(比如加法的操作)，chip-8编码这些到操作码本身。因为这样，所有的chip-8操作码是精确的2个字节，它比一些其他系统更大，但是整个指令存储在这些两个字节里面，但是同时代系统可能消耗在每周期1到3个字节之间。

Each opcode is encoded differently, but fortunately since all instructions consume two bytes, the fetch operation is the same for all of them, and implemented as such:

每个操作码被不同的编码，但是，幸运的是，因为所有的指令消耗2个字节，然后遍历所有的操作码是相同的，像这样实现:

```rust
fn fetch(&mut self) -> u16 {
    let higher_byte = self.ram[self.pc as usize] as u16;
    let lower_byte = self.ram[(self.pc + 1) as usize] as u16;
    let op = (higher_byte << 8) | lower_byte;
    self.pc += 2;
    op
}
```

This function fetches the 16-bit opcode stored at our current Program Counter. We store values in RAM as 8-bit values, so we fetch two and combine them as Big Endian. The PC is then incremented by the two bytes we just read, and our fetched opcode is returned for further processing.

这个函数遍历由我们当前程序计数器指向的存储的操作码。我们存储作为8位值在RAM里面，所以，我们遍历两个，然后组合他们作为大端格式。这个PC然后按照我们将要读的两个字节来增加,我们遍历的操作码作为将来处理而返回。

## Timer Tick
## 时钟滴答
The Chip-8 specification also mentions two special purpose *timers*, the Delay Timer and the Sound Timer. While the `tick` function operates once every CPU cycle, these timers are modified instead once every frame, and thus need to be handled in a separate function. Their behavior is rather simple, every frame both decrease by one. If the Sound Timer is set to one, the system will emit a 'beep' noise. If the timers ever hit zero, they do not automatically reset; they will remain at zero until the game manually resets them to some value.

chip-8类型也提到两个专有目的的*timers*,延迟时钟和声音时钟。每个cpu周期`tick`函数操作一次，相应的每帧这些时间被修改一次，因此需要再一个分开的函数来处理。他们的行为相当简单，每帧它们两个减少1.如果声音时钟被设置为1，这个系统将发出一个'beep'声音。如果这个时钟一旦击中0，他们不会自动重设;他们将维持在0知道游戏手动重设他们到相同值。

```rust
pub fn tick_timers(&mut self) {
    if self.dt > 0 {
        self.dt -= 1;
    }

    if self.st > 0 {
        if self.st == 1 {
            // BEEP
        }
        self.st -= 1;
    }
}
```

Audio is the one thing that this guide won't cover, mostly due to increased complexity in getting audio to work in both our desktop and web browser frontends. For now we'll simply leave a comment where the beep would occur, but any curious readers are encouraged to implement it themselves (and then tell me how they did it).
音频是这个向导将不会覆盖，绝大部分原因是在获取音频增加复杂性在我们桌面应用和web浏览器的前段。为此，我们将简单留下注释在beep将发生的地方，但是一些感兴趣的读者建议去实现他们(然后告诉我他们怎样处理它的)
\newpage
