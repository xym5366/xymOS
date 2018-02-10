# xymOS系统制作
自制系统，空余时间少，少量代码
主要以nasm代码为主，版本暂时忘了，有空在详细介绍。
* 首先在dos下使用命令`nasm boot.asm -o boot.bin`编译boot.asm文件为boot.bin纯二进制指令代码。
  * `$`为地址计数器，每一条汇编指令使得或数据定义指令都会在给`$`加上自身要用的内存单元数量 `org` 用来重置`$`的值 `org X`即表示重置`$=X`，于是下一个内存单元的地址将是`X`
  * `$-$$`得出此前占用的字节存储容量（由此判断`$$`为代码段起始处，证明`$$`的确为上一段的起始地址，懒的看手册，从csdn看到的），再用510减得出后面容量并填充0，留两字节填充`aa55`
  * `$`准确讲是模拟当前ip寄存器值，`org`是指定此处ip寄存器值。原因是磁盘里的机器指令并没有任何状态，拷贝到内存里才有实际状态，这两个就是来模拟将要在内存里状态的作用。
- masm 实模式进保护模式代码(基于dos7.1保护模式操作系统masm5.0代码)
  * 注意，加载内核到内存中，此内核程序必须要符合实际内存地址（就是要拷贝到内存的地址，不可以随意拷贝到任意内存地址，必须固定和代码org、$、$$吻合）
  * 所以masm语法具体不详，此处org、$、$$语法为nasm语法。(nasm较早版本，具体版本忘了;)有时间再看看，最近很忙)

```
33C08EC08ED88ED0BC007CFC8BF4BF0006B90001F2A5EA670600008BD558A24F073C357423B410F6E405AE048BF0807C04007444807C0405743EC60480E8DA008A74018B4C02EB08E8CF00B9010032D1BB007CB80102CD13721E81BFFE0155AA7516EA007C000080FA817402B2808BEA4280F2B388164107BFBE07B90400C60634073132F6882D8A45043C0074233C05741FFEC6BE3107E87100BE4F0746468B1C0AFF7405327D0475F38DB77B07E85A0083C710FE063407E2CB803E750402740BBE42070AF6750ACD18EBACBE3107E83900E8360032E4CD1A8BDA83C360B401CD16B400750BCD1A3BD372F2A04F07EB0ACD168AC43C1C74F304F63C3172D63C3577D250BE2F07BB1B0653FCAC50247FB40ECD1058A88074F2C356B80103BB0006B9010032F6CD135EC6064F073FC30D8A0D0A4635202E202E202EA06469736B20320D0A0A44656661756C743A204631A00001000400060307070A0A630E640E6514801981198219831E9324A52B9F2F75335233DB36403BF24100446FF3487066F34F73B2556E69F84E6F76656CEC4D696E69F84C696E75F8416D6F6562E1467265654253C4425344E9506369F84370ED56656E69F8446F737365E33FBF800101000103512D11000000F44F000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000055AA
```
发一段一个DOS的启动扇区

反汇编以上16进制代码：
```
0000:7c00: xor ax,ax ;33c0
0000:7c02: mov es,ax ;8ec0
0000:7c04: mov ds,ax ;8ed8
0000:7c06: mov ss,ax ;8ed0
0000:7c08: mov sp,0x7c00 ;bc007c
0000:7c0b: cld ;fc
0000:7c0c: mov si,sp ;8bf4
0000:7c0e: mov di,0x0600 ;bf0006
0000:7c11: mov cx,0x0100 ;b90001
0000:7c14: repne movsw word ptr es:[di], word ptr ds:[si] ;f2a5
... ; 重复执行0000:7c14移动0000:7c00数据到0000:0600内存地址 指令
0000:7c16: jmpf 0x0000:0667 ;ea67060000
....
0000:0667: cmp dl, 0x81 ;80fa81 启动扇区偏移第67字节处
0000:066a: jz .+2 (0x0000066e) ;7402
0000:066c: mov dl,0x80 ;b280
0000:066e: mov bp,dx ;8bea
0000:0670: inc dx ;42
0000:0671: xor dl,0xb3 ;80f2b3
0000:0674: mov byte ptr ds:0x0741, dl ;88164107
0000:0678: mov di,0x07be ;bfbe07
0000:067b: mov cx,0x0004 ;b90400
0000:067e: mov byte ptr ds:0x0734, 0x31 ;c606340731
0000:0683: xor dh,dh ;32f6
0000:0685: mov byte ptr ds:[di], ch ;882d
0000:0687: mov al, byte ptr ds:[di+4] ;8a4504
0000:068a: cmp al,0x00 ;3c00
0000:068c: jz .+35 (0x000006b1) ;7423
0000:068e: cmp al, 0x05 ;3c05
0000:0690: jz .+31 (0x000006b1) ;741f
0000:0692: inc dh ;fec6
0000:0694: mov si,0x0731 ;be3107
0000:0697: call .+113 (0x0000070b) ;e87100
...
0000:070b: cld ;fc
0000:070c: lodsb al, byte ptr ds:[si] ;ac
0000:070d: push ax ;50
0000:070e: and al,0x7f ;247f
0000:0710: mov ah, 0x0e ;b40e
0000:0712: int 0x10 ;cd10
...
c000:0152: pushf ;9c
;今天先干到这2018/1/18 13:40
```
