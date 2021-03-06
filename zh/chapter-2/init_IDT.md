#【实现】初始化中断门描述符表

ucore操作系统如果要正确处理各种不同的中断事件，就需要安排应该由哪个中断服务例程负责处理特定的中断事件。系统将所有的中断事件统一进行了编号（0～255），这个编号称为中断号或中断向量。

为了完成中断号和中断服务例程起始地址的对应关系，首先需要建立256个中断处理例程的入口地址。为此，通过一个 C程序 tools/vector.c 生成了一个文件vectors.S，在此文件中的 \__vectors地址处开始处连续存储了256个中断处理例程的入口地址数组，且在此文件中的每个中断处理例程的入口地址处，实现了中断处理过程的第一步初步处理。

有了中断服务例程的起始地址，就可以建立对应关系了，这部分的实现在trap.c文件中的idt_init函数中实现：

	//全局变量：中断门描述符表

    static struct gatedesc idt[256] = {{0}};
    ……
	void idt_init(void) {
    
	//保存在vectors.S中的256个中断处理例程的入口地址数组

        extern uint32_t __vectors[];
        int i;
      
    //在中断门描述符表中通过建立中断门描述符，其中存储了中断处理例程的代码段GD_KTEXT和偏移量\__vectors[i]，特权级为DPL_KERNEL。这样通过查询idt[i]就可定位到中断服务例程的起始地址。

        for (i = 0; i < sizeof(idt) / sizeof(struct gatedesc); i ++) {
            SETGATE(idt[i], 0, GD_KTEXT, __vectors[i], DPL_KERNEL);
        }
      
	//建立好中断门描述符表后，通过指令lidt把中断门描述符表的起始地址装入IDTR寄存器中，从而完成中段描述符表的初始化工作。

    	lidt(&idt_pd);
    }

