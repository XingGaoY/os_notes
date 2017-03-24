# proj1
## sign.c
扩展文件大小到512k，同时修改文件结尾为0x55AA; 

连接好bin/bootblock之后，将其作为第一个参数运行sign.c，用来产生一个主引导扇区；
```
13      Get the info of a file
23-25   creat a new buffer and read file1 into it
36      write the modified file ito the second file
```
## bootasm.S
启动保护模式，跳转到c语言，相关内容：
- [ucore bootloader分析](http://www.cnblogs.com/maruixin/p/3175894.html)
- [xv6 bootloader分析](http://blog.csdn.net/qq_25426415/article/details/54583835)
```
13      The main entry of bootasm.S
15      Set the mode to 16 bit
29-43   Activate A20. Similar with xv6: test the port 0x60 & 0x64 free, and send 0xdf & 0xd1 respectively
49      Load the temp GDT
56      ljmp to C code with mode 32
69-71   !!Set up stack pointer and call bootmain
```
Check the disassembly code we can find that the <start> section start with **0x7c00**, which is set during compile
# proj2
proj2的bootloader过大，为了能够运行，我删除了有关```cons_puts```的实现（事实上也不需要这些内容）。

# proj4
- 事实上，段选择子的保存类似一张二级表：GDT保存各个进程的LDT，LDT进一步保存每个段的起始地址。GDT保存了指向LDT段的选择子。 
- IDT找到对应的GDT中的段选择子，获得基址后和偏移相加，得到对应的中断服务例程地址。
