---
layout: default
---


# 模拟器和游戏项目
**最新公告**
```
 > 新增教学用的模拟器(cpu:x86,mips,arm等,操作系统,网络)(2025-11-22)
 > 游戏开发分享计划sfml系列:
    > sfml game development by example(2025-11-18)
    > sfml essentials
    > sfml blueprint
    > master sfml
 > pc模拟器开发分享:
    > 8086模拟器（2025-11-18)
    > ibm-pc 5150
    > 80386模拟器
    > bochs模拟器和linux 0.11
    > qemu模拟器
 > 实现完整模拟器计划:
    > gb和gba模拟器
 > 添加一些复古计算机链接(2025-11-17)
```
- [resource](#1-resouce)
- [books](#2-books)
- [emulator doc](#3-emulator-doc)
- [开源项目](#4-开源项目)
- [call me](#5-call-me)
- [待办建议](#6-待办建议)



<!-- fence:start -->
## 1. **resource**  
### 1.1 general
- [知乎](https://www.zhihu.com/people/jim-79-39-91)
- [B站](https://space.bilibili.com/3493279404395296) 
- [emulator study path](/study_path.md) 
- [demo](/docs/demo/index.html)
- [Emu-Docs](https://github.com/shonumi/Emu-Docs) 
- [awesome-emu-resources](https://github.com/marethyu/awesome-emu-resources)
- [EE2 Computing Course main page](http://www.ee.ic.ac.uk/pcheung/teaching/ee2_computing/)
- [HOWTO: Writing a Computer Emulator](http://fms.komkon.org/EMUL8/HOWTO.html) 
- [CSEE 4840 Embedded System Design](https://www.cs.columbia.edu/~sedwards/classes/2016/4840-spring/)
- [some cpu and emulator](https://www.zophar.net/)
- [Instruction-Set Simulators for Real and Virtual Machines](http://www.xsim.com/) 
- [Study of the techniques for emulation programming](http://www.xsim.com/papers/Bario.2001.emubook.pdf)
- [Awesome Xbox Development](https://xbox1.dev/)
* 复古计算机
  * [复古计算机](https://www.retromuseum.org/)
  * [古董电脑室](https://pengan1987.github.io/)
  * [中文电脑历史研究馆](https://rpc.huijiwiki.com/wiki/%E9%A6%96%E9%A1%B5)
  * [x86复古在线模拟器](https://copy.sh/v86/)
  * [v86另外一个在线模拟器](https://xb6.cn/tools/v86/)
  * [mit相关历史平台研究资料](https://mitpress.mit.edu/series/platform-studies/)
  * [老电脑博物馆](https://www.oldcomputermuseum.com/default.htm)
  * [旧电脑1](https://oldcomputer.info/index.htm)
  * [旧电脑2](https://oldcomputers.net/indexwp.html)
  * [一些比较老的计算机相关书籍](http://www.rsp-italy.it/IT/Books/)
* 游戏:

  * [游戏开发学习路线图](https://miloyip.github.io/game-programmer/)
  * [游戏开发路线图一些书籍](https://github.com/kurong00/GameProgramBooks)         
  * [游戏开发书籍](https://github.com/BuggerBag/Game-Driven-Development-)
  * [游戏开发书籍](https://github.com/AlphaStarX/GameProgrammingBooks)
  * [游戏引擎](https://brodydoesart.github.io/GameEngineBook/)
   
### 1.2 cpu
### 1.2.1 6502 
 * [online 6502 disassembler](https://jborza.com/post/2021-06-08-6502-disassembler/)
### 1.2.2 x86
 * [documentation for emu8086](https://yassinebridi.github.io/asm-docs/help.html)
 * [mit 6.828课程一些参考资料](https://pdos.csail.mit.edu/6.828/2018/reference.html)
 * [bochs 一些参考资料](https://bochs.sourceforge.io/techdata.html)
 * [unix操作系统注解作者主页](http://www.lemis.com/grog/Documentation/Lions/)
 * [早期一些unix相关版本和源码](https://minnie.tuhs.org/cgi-bin/utree.pl)
### 1.2.3 教学用模拟器
 * [cpu-os:仿真cpu和os，网络](https://teach-sim.com/)
 * [cpulator:一个在线学习arm,mips等汇编的在线模拟器学习工具](https://cpulator.01xz.net/)
<!-- fence -->
## 2. **books**
- [游戏模拟器开发教程(初级版)](docs/game_emulator_develop(primary).html)
- [chip8模拟器开发(入门)](docs/step_by_step_development_chip8_emulation.html)
```
  书名:<<chip8模拟器开发入门>>
  英文名:step by step development chip8 emulation
  当前状态:完成初稿(2025-11-1)
  电子版价格:￥10,粉丝价:￥5
  纸质:待定
  抢先看当前完成部分:加入模拟器开发会员群 205401359(qq) 
  付款方式:本页面最下面有二维码，扫码完成，备注写:购买<<chip8模拟器开发入门>>电子版.如果是粉丝，麻烦提供b站等昵称。
  关于退款:如果对本书不满意，或者没兴趣，可以在付款后一个月内私我进行退款操作，我将按在原来付款帐号返还。
```
* **书籍**:
  * **编写中**
    - [怎样编写chip8编译器]
    - [一步步学习汇编--6502]

  * **计划中**
    - [一步步学习汇编--arm] 
    - [一步步学习汇编--mips]
    - [一步步学习汇编--8086]
    - [一步步学习汇编--80386]
    - [一步步学习汇编--z80]
    - [一步步学习汇编--pdp11]
    - [gb模拟器开发入门]
    - [gb模拟器开发中级-全指令集覆盖]
    - [2d游戏开发入门]

<!-- fence -->
## 3. **emulator doc**  

* [-][chip8](docs/chip8/index.html)
* [x][game boy](docs/gb/index.html)
* [x][nes](docs/nes/index.html)
* [x][snes](docs/snes/index.html)
* [x][gba](docs/gba/index.html)
* [x][nds](docs/nds/index.html)
* [x][n64](docs/n64/index.html)
* [x][ps1](docs/ps1/index.html)
* [x][xbox](docs/xbox/index.html)

<!-- fence -->
## 4. **开源项目**
- [no]()

## 5. **游戏**
 * [xbox game](https://myrient.erista.me/files/Redump/Microsoft%20-%20Xbox/)
 * 关于游戏完整视频:
   ```
   · sdl游戏开发入门(单个视频:1元/个，全部:20/专辑)
   · sfml游戏开发入门(单个视频:1元/个，全部:30/专辑)
   · sfml使用例子进行游戏开发(单个视频:1元/个，全部:未定)
   ```
<!-- fence -->
## 6. **call me**

- 模拟器爱好者交流群:193548232(qq)
```
  · 进入本群需要每个成员至少完成一个可以独立运行的游戏rom
```
- 模拟器和游戏开发会员群:205401359(qq)和微信
  <img src="webchat_group.jpg" width=300 height=200 />
```
  · 会员群只接受人员
  · 购买书籍
  · 购买学习指导服务
  · 模拟器使用者付费者
  · 游戏开发学习完整视频和指导
```
- 会费:18.8/月,188/年
```
    · 游戏完整教学和实验视频(免费)
    · 支持基本游戏或者模拟器开发指导
    · 协助一些开发问题分析和建议
    · 其他:后面补充
```
- 邮箱:370858342@qq.com
- webchat:jinjiacun123
- 如果这些技术对你有所帮助，对制作者进行捐助以此鼓励。

![Initial file structure](pay.jpg)


## 7. **待办建议**
- [建议](docs/advance.html)
<!-- fence:end -->












