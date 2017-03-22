# 了解x86保护模式中的特权级
- CPU在以下时刻检查特权级：
	- 访问数据段
	- 访问页
	- 进入终端服务例程
检查失败，进入page fault 

- 段选择子
位于段寄存器中。
	- 最后两位：RPL数据段（ES），CPL代码段（CS），同段描述符，中断门或陷入门中的DPL进行比较
		- 访问门（中断陷入异常）：CPL <= DPL[gate] & CPL >= DPL[seg] *访问本身不可访问的地址*
		- 访问段：MAX(CPL, RPL) <= DPL[seg]

# 了解特权级切换过程
- ring0 -> ring3
	- **内核**产生中断
	- 内核态ring0中压入恢复程序的信息：SS(RPL = 3), ESP, EFLAGES, CS, EIP, Error Code
	- **构造**一个特殊的栈，来到ring3，将SS，CS设置为CPL\RPL，通过IRET弹出栈，完成对于寄存器的赋值
- ring3 -> ring0
	- **用户态**产生中断
	- 保存信息
	- 构造栈，不需要SS，ESP，将CPL转换为0，IRET返回即可。
- TSS格式
中断之后跳转的信息保存在内存中的任务状态段（TSS）中，全局描述符表中存有一个TSS Descriptor。 
通过Task Register缓存信息。
- 建立TSS
	- Allocate TSS memory
	- Init TSS
	- Fill TSS descriptor in GDT
	- Set TSS selector

# 了解段\页表
- MMU段机制描述
	- 段寄存器：CS、ES、SS、DS、FS、GS
	- Base Address + addr 得到线性或者物理地址
	- GDT在内存中，访问会很慢，段选择子会有一段隐藏在CPU中，加快访问速度
- 建立GDT
	- entry.S 将基地址-0xC0000000
**Some Questions Left!!//TODO**

#了解uCore建立段页表
- uCore中是二级页表：
	- 10位Dir查找PDE项（PDE起始地址保存在CR3中），向**左移**12位，找到PTE起始地址。
	- 10位Table查找PTE项，也同样左移12位，得到页表的起始地址
	- 加上12位offset得到**线性地址**
- 页表项，保存一些位
	- R/W：是否可写
	- U/S：特权级
- 使能页机制 
在保护模式中将CR0的bit31（PG）置零
- 建立页表
	- 分配一个页用来建立目录
	- 清理分配的页用来初始化
	- 建立对应关系，将0xC0000000-0xF8000000（va）映射到0x00000000-0x38000000（pa）
	- 将0x0000000-0x00100000做一个对等映射
	- 使能页机制，更新GDT
	- 取消第四步的对等映射（为什么作这几步）
