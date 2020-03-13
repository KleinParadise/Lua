新建TestGlobal.lua文件包含以下代码
```lua
var = 12345
var2 = 45678
local function fun()
  local tempVar = var + var2
  return tempVar
end
```
使用ChunkSpy.lua对TestGlobal.lua反编译得到以下汇编代码
```
; source chunk: luac.out
; x86 standard (32-bit, little endian, doubles)

; function [0] definition (level 1)
; 0 upvalues, 0 params, 2 stacks
.function  0 0 2 2
.local  "fun"  ; 0
.const  "var"  ; 0
.const  12345  ; 1
.const  "var2"  ; 2
.const  45678  ; 3

; function [0] definition (level 2)
; 0 upvalues, 0 params, 2 stacks
.function  0 0 0 2
.local  "tempVar"  ; 0
.const  "var"  ; 0
.const  "var2"  ; 1
; (3)  local function fun()
; (4)  	local tempVar = var + var2
[1] getglobal  0   0        ; var
[2] getglobal  1   1        ; var2
[3] add        0   0   1  
; (5)  	return tempVar
[4] return     0   2      
; (6)  end
[5] return     0   1      
; end of function

; (1)  var = 12345
[1] loadk      0   1        ; 12345
[2] setglobal  0   0        ; var
; (2)  var2 = 45678
[3] loadk      0   3        ; 45678
[4] setglobal  0   2        ; var2
; (6)  end
[5] closure    0   0        ; 0 upvalues
[6] return     0   1      
; end of function
```
24-39行是函数fun的指令chunk,其他行为TestGlobal文件的指令chunk

首先分析TestGlobal文件的指令chunk,以下为除去函数fun的汇编代码
```lua
; source chunk: luac.out
; x86 standard (32-bit, little endian, doubles)

; function [0] definition (level 1)
; 0 upvalues, 0 params, 2 stacks
.function  0 0 2 2
.local  "fun"  ; 0
.const  "var"  ; 0
.const  12345  ; 1
.const  "var2"  ; 2
.const  45678  ; 3

; (1)  var = 12345
[1] loadk      0   1        ; 12345
[2] setglobal  0   0        ; var
; (2)  var2 = 45678
[3] loadk      0   3        ; 45678
[4] setglobal  0   2        ; var2
; (6)  end
[5] closure    0   0        ; 0 upvalues
[6] return     0   1      
; end of function
```
62行,fun函数会被创建在0号寄存器位置
63行,字符串var会被存储在常量数组索引0的位置
64行,12345会被存储在常量数组索引1的位置
65行,字符串var2会被存储在常量数组索引2的位置
66行,45678会被存储在常量数组索引3的位置

69行,生成指令loadk,从该chunk的Proto结构保存的常量数组k中取出索引为1的数据,赋值给0号寄存器
70行,生成指令setglobal,该chunk的Proto结构保存的常量数组k中取出索引为0的数据key 将0号寄存器的值赋值给Gbl[key](Gbl为全局符号表,key为全局变量名,value为全局变量的值)
72-73行与上述两步相同,只是指令操作的索引不同
75行,生成指令closure,生成func函数对象并放置在0号寄存器

接下来分析函数func的汇编代码
```lua
; function [0] definition (level 2)
; 0 upvalues, 0 params, 2 stacks
.function  0 0 0 2
.local  "tempVar"  ; 0
.const  "var"  ; 0
.const  "var2"  ; 1
; (3)  local function fun()
; (4)  	local tempVar = var + var2
[1] getglobal  0   0        ; var
[2] getglobal  1   1        ; var2
[3] add        0   0   1  
; (5)  	return tempVar
[4] return     0   2      
; (6)  end
[5] return     0   1      
; end of function
```
95行，tempVar的值放在0号寄存器
96行,字符串var会被存储在func函数Proto结构中常量数组k的索引为0位置
97行,字符串var2会被存储在func函数Proto结构中常量数组k的索引为1位置

100行,生成指令getglobal,从func函数Proto结构中常量数组k的索引为0位置取出key值var,通过key从全局符号表Gbl从取出全局变量的值赋值给0号寄存器
101行,生成指令getglobal,从func函数Proto结构中常量数组k的索引为1位置取出key值var2,通过key从全局符号表Gbl从取出全局变量的值赋值给1号寄存器
102行,生成指令add,将0号寄存器与1号寄存器值相加,并放置到0号寄存器(代码层面赋值给tempVar)
104行,生成指令return,从0号寄存器地址返回,返回值的个数为2-1个
