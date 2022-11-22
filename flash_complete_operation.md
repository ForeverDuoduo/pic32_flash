# flash_complete_operation

```c
static inline int flash_complete_operation(void)
{
	u32 tmp;

	tmp = readl(&nvm_regs_p->ctrl.raw);
	if (tmp & NVM_WRERR) {	//WRERR:写错误位
		printf("Error in Block Erase - Lock Bit may be set!\n");
		flash_initiate_operation(NVMOP_NOP);
		return ERR_PROTECTED;	//ERR_PROTECTED:受保护
	}

	if (tmp & NVM_LVDERR) {	//LVDERR:低电压检测错误位 
		printf("Error in Block Erase - low-vol detected!\n");
		flash_initiate_operation(NVMOP_NOP);
		return ERR_NOT_ERASED;//NOT_ERASED：未擦除
	}

	/* disable flash write or erase operation */
	writel(NVM_WREN, &nvm_regs_p->ctrl.clr);

	return ERR_OK;
}
```

完整的操作(?)

控制寄存器(?)

> 第一步：判断tmp是否为NVM_WRERR
> 其中tmp为nvm_regs_p->ctrl.raw，如果是，说明Lock Bit may be set
> return ERR_PROTECTED 表示扇区受保护，不允许读写
>
> 第二步：判断是否为NVM_LVDERR
> 如果是，说明low-vol detected，即目前存储空间不足
> return ERR_NOT_ERASED 说明扇区擦除失败
>
> ​      Error in Block Erase说明错误都发生在扇区擦除
>
> 第三步：在前两步基础上说明扇区擦除没问题，那就可以write了
> 写好后会return OK，表示flash_complete_operation操作的结果



> WR:写控制位
>  该位不可清零，仅当 WREN = 1 并执行解锁序列时可置 1。
>  1 = 启动闪存操作
>  0 = 闪存操作已完成或无效
>
> WREN:写使能位
>  1 = 使能对 WR 位的写操作，并禁止对 SWAP 或 PFSWAP 位以及 BFSWAP 和 NVMOP\<3:0> 位的写操作
> 0 = 禁止对 WR 位的写操作，并使能对 SWAP 或 PFSWAP 位以及 BFSWAP 和 NVMOP\<3:0> 位的写操作 WRERR:写错误位
>
> 该位只能通过设置 NVMOP\<3:0> 位 = 0000 (NOP)并启动闪存操作清零。 
>  1 = 编程或擦除序列未成功完成
>  0 = 编程或擦除序列正常完成
>
> LVDERR:低电压检测错误位
>
> 该位只能通过设置 NVMOP\<3:0> 位 = 0000 (NOP)并启动闪存操作清零。
>  1 = 编程或擦除操作时检测到低电压条件 (如果 WRERR 置 1，则数据可能损坏) 0 = 编程或擦除操作时未发生低电压条件
