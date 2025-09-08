\newpage

# An Introduction to Video Game Emulation with Rust
# 介绍使用rust开发的视频游戏模拟器
Developing a video game emulator is becoming an increasingly popular hobby project for developers. It requires knowledge of low-level hardware, modern programming languages, and graphics systems to successfully create one. It is an excellent learning project; not only does it have clear goals, but it also very rewarding to successfully play games on an emulator you've written yourself. I am still a relatively new emulation developer, but I wouldn't have been able to reach the place I am now if it weren't for excellent guides and tutorials online. To that end, I wanted to give back to the community by writing a guide with some of the tricks I picked up, in hopes it is useful for someone else.

对于开发人员来说，开发一个视频游戏模拟器已经变成一个逐渐流行的爱好型项目。它需要底层硬件，现代编程语言和图形系统来成功创建它们。它是一个非常优秀的项目;不仅仅有个清晰的目标完成它，而且可以很大的回报，用自己写的模拟器来玩游戏。我仍然是一个相关领域的模拟器开发新手，但是如果没有优秀的指导和在线手册，我将无法达到现在的水平。为此，我想通过我自己收集的一些技术编写一个导引回馈给社区，希望对其他一些人有所帮助.

## Intro to Chip-8
## 介绍Chip-8
Our target system is the [Chip-8](https://en.wikipedia.org/wiki/CHIP-8). The Chip-8 has become the "Hello World" for emulation development of a sort. While you might be tempted to begin with something more exciting like the NES or Game Boy, these are a level of complexity higher than the Chip-8. The Chip-8 has a 1-bit monochrome display, a simple 1-channel single tone audio, and only 35 instructions (compared to ~500 for the Game Boy), but more on that later. This guide will cover the technical specifics of the Chip-8, what hardware systems need to be emulated and how, and how to interact with the user. This guide will focus on the original Chip-8 specification, and will not implement any of the many proposed extensions that have been created, such as the Super Chip-8, Chip-16, or XO-Chip; these were created independently of each other, and thus add contradictory features.

我们的目标系统是[Chip-8].这个chip8在模拟器开发序列中成为了模拟器开发的"hello world".当你可能对于像NES或者Game Boy更精彩的模拟器有兴趣时，这些相对于Chip-8是一个更高复杂层次的。Chip-8有一个1位单色显示，一个单一声道，仅有35个指令(相比于Game Boy接近500个)，但是后面更多。这个导论将覆盖chip-8的专门技术，哪些硬件系统需要被模拟，和怎样和用户交互。这个向导将焦点在专门的chip-8的组成，将不会实现一些已经创建的被建议的扩展，比如超级Chip-8,Chip-16,或者XO-Chip;这些每个独立的被创建,因此会增加冲突特性。
## Chip-8 Technical Specifications
## Chip-8专有技术
- A 64x32 monochrome display, drawn to via sprites that are always 8 pixels wide and between 1 and 16 pixels tall
- 一个64x32单色显示，绘制经由8个像素宽和在1和16像素高的精灵。
- Sixteen 8-bit general purpose registers, referred to as V0 thru VF. VF also doubles as the flag register for overflow operations
- 16个8位一般目的寄存器，参考如V0到VF.
- 16-bit program counter
- 16位程序计数器
- Single 16-bit register used as a pointer for memory access, called the *I Register*
- 单独16位寄存器作为内存访问的指针使用，叫作"I寄存器"
- An unstandardised amount of RAM, however most emulators allocate 4 KB
- 一个非标准化的RAM大小，但是大部分模拟器分配4KB空间
- 16-bit stack used for calling and returning from subroutines
- 16位栈被用于访问和子实例返回
- 16-key keyboard input
- 16个键的键盘输入
- Two special registers which decrease each frame and trigger upon reaching zero:
    - Delay timer: Used for time-based game events
    - Sound timer: Used to trigger the audio beep
- 两个专门寄存器，用于每帧的减少和上发到0
    - 延时时钟:用于基于时间的游戏事件
    - 声音时钟:用于触发beep声音
## Intro to Rust
Emulators can be written in nearly any programming language. This guide uses the [Rust programming language](https://www.rust-lang.org/), although the steps outlined here could be applied to any language. Rust offers a number of great advantages; it is a compiled language with targets for major platforms and it has an active community of external libraries to utilize for our project. Rust also supports building for [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly), allowing us to recompile our code to work in a browser with a minimal amount of tweaking. This guide assumes you understand the basics of the Rust language and programming as a whole. I will explain the code as we go along, but as Rust has a notoriously high learning curve, I would recommend reading and referencing the excellent [official Rust book](https://doc.rust-lang.org/stable/book/title-page.html) on any concepts that are unfamiliar to you as the guide progresses. This guide also assumes that you have Rust installed and working correctly. Please consult the [installation instructions](https://www.rust-lang.org/tools/install) for your platform if needed.

## What you will need
## 你将需要什么
Before you begin, please ensure you have access to or have installed the following items.

在开始之前，请确保你已经访问或者已经安装了下面项.
### Text Editor
### 文本编辑器
Any text editor can be used for the project, but there are two I recommend which offer features for Rust like syntax highlighting, code suggestions, and debugger support.

这个项目需要一些文本编辑编辑器，但是我有两个推荐的提供rust语法高亮，代码建议，和支持调试。
- [Visual Studio Code](https://code.visualstudio.com/) is the editor I prefer for Rust, in combination with the [rust-analyzer](https://rust-analyzer.github.io/) extension.

- While JetBrains does not offer a dedicated Rust IDE, there is a Rust extension for many of its other products. The [extension](https://intellij-rust.github.io/) for [CLion](https://www.jetbrains.com/clion/) has additional functionality that the others do not, such as integrated debugger support. Keep in mind that CLion is a paid product, although it offers a 30 day trial as well as extended free periods for students.

If you do not care for any of these, Rust syntax and autocompletion plugins exist for many other editors, and it can be debugged fairly easily with many other debuggers, such as gdb.

如果你不关心这些，Rust语法和作为其他编辑器的自动生成挂件，它能用一些其他调试工具，比如gdb.
### Test ROMs
### 测试roms
An emulator isn't much use if you have nothing to run! Included with the source code for this book are a number of commonly distributed Chip-8 programs, which can also be found [here](https://www.zophar.net/pdroms/chip8/chip-8-games-pack.html). Some of these games will be shown as an example throughout this guide.

### Misc.
### 资源
Other items that may be helpful as we progress:
其他一些项目可能是有用的:
- Please refresh yourself with [hexadecimal](https://en.wikipedia.org/wiki/Hexadecimal) notation if you do not feel comfortable with the concept. It will be used extensively throughout this project.

- Chip-8 games are in a binary format, it is often helpful to be able to view the raw hex values as you debug. Standard text editors typically don't have support for viewing files in hexadecimal, instead a specialized [hex editor](https://en.wikipedia.org/wiki/Comparison_of_hex_editors) is required. Many offer similar features, but I personally prefer [Reverse Engineer's Hex Editor](https://github.com/solemnwarning/rehex).

Let's begin!

\newpage
