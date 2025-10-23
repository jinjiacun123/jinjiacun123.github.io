大纲:
# 导读
# 第一章 我不想知道太多，也记不住那么多概念
## 1.1 我想开发chip8模拟器，那么最简单的rom是啥样？
## 2.2 让我的最简单rom跑起来，完成我的chip8模拟器的hello world
# 第二章 我想要加点功能
## 2.1 谁来帮助我加功能 -- 指令
## 2.2 怎样算1+1?
## 2.3 Chip8指令集
### 2.3.1 chip8指令-(00E0 - CLS)
### 2.3.2 chip8指令-(00EE - RET)
### 2.3.3 chip8指令-(0nnn - SYS addr)
### 2.3.4 chip8指令-(1nnn - JP addr)
### 2.3.5 chip8指令-(2nnn - CALL addr)
### 2.3.6 chip8指令-(3xkk - SE Vx, byte)
### 2.3.7 chip8指令-(4xkk - SNE Vx, byte)
### 2.3.8 chip8指令-(5xy0 - SE Vx, Vy)
### 2.3.9 chip8指令-(6xkk - LD Vx, byte)
### 2.3.10 chip8指令-(7xkk - ADD Vx, byte)
### 2.3.11 chip8指令-(8xy0 - LD Vx, Vy)
### 2.3.12 chip8指令-(8xy1 - OR Vx, Vy)
### 2.3.13 chip8指令-(8xy2 - AND Vx, Vy)
### 2.3.14 chip8指令-(8xy3 - XOR Vx, Vy)
### 2.3.15 chip8指令-(8xy4 - ADD Vx, Vy)
### 2.3.16 chip8指令-(8xy5 - SUB Vx, Vy)
### 2.3.17 chip8指令-(8x06 - SHR Vx)
### 2.3.18 chip8指令-(8xy7 - SUBN Vx, Vy)
### 2.3.19 chip8指令-(8x0E - SHL Vx)
### 2.3.20 chip8指令-(9xy0 - SNE Vx, Vy)
### 2.3.21 chip8指令-(Annn - LD I, addr)
### 2.3.22 chip8指令-(Bnnn - JP V0, addr)
### 2.3.23 chip8指令-(Cxkk - RND Vx, byte)
### 2.3.24 chip8指令-(Dxyn - DRW Vx, Vy, nibble)
### 2.3.25 chip8指令-(Ex9E - SKP Vx)
### 2.3.26 chip8指令-(ExA1 - SKNP Vx)
### 2.3.27 chip8指令-(Fx07 - LD Vx, DT)
### 2.3.28 chip8指令-(Fx0A - LD Vx, K)
### 2.3.29 chip8指令-(Fx15 - LD DT, Vx)
### 2.3.30 chip8指令-(Fx18 - LD ST, Vx)
### 2.3.31 chip8指令-(Fx1E - ADD I, Vx)
### 2.3.32 chip8指令-(Fx29 - LD F, Vx)
### 2.3.33 chip8指令-(Fx33 - LD B, Vx)
### 2.3.34 chip8指令-(Fx55 - LD [I], Vx)
### 2.3.35 chip8指令-(Fx65 - LD Vx, [I])
# 第三章 chip8架构补充
## 3.1 关于cpu
## 3.2 关于内存
## 3.3 关于键盘
## 3.4 关于时钟
# 第四章 怎样和宿主机对接?
## 4.1 我们选择哪种第三方库？
## 4.2 如何对接宿主机图像部分
## 4.3 如何对接宿主机键盘部分
## 4.4 如何对接宿主机声音部分
# 第五章 关于设计一些rom
## 5.1.0 我想看点到东西
## 5.1.1 我有一个idea,很简单，我想在屏幕左上角演示一个点
## 5.1.2 怎样把我想法转化为功能模块
## 5.1.3 好吧，好像有些多和重复了，能不能再精简下
## 5.2.0 我感觉我有点了解了，是不是能加个功能
## 5.2.1 我想数字显示出来
## 5.2.2 是不是来个倒计时呢
## 5.3.0 我可以进行交互么(键盘)？
## 5.3.1 我想做个10以内的加减法测试？
## 5.3.2 或许我可以移动我的小宝贝
## 5.4.0 或许可以给我点提示(声音)？
## 5.4.1 我都撞上了，你给我点反应好吧
# 参考资料
# 附录
