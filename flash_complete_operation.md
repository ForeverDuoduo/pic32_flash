# flash_complete_operation

```c
static inline int flash_complete_operation(void)
{
	u32 tmp;

	tmp = readl(&nvm_regs_p->ctrl.raw);
	if (tmp & NVM_WRERR) {
		printf("Error in Block Erase - Lock Bit may be set!\n");
		flash_initiate_operation(NVMOP_NOP);
		return ERR_PROTECTED;
	}

	if (tmp & NVM_LVDERR) {
		printf("Error in Block Erase - low-vol detected!\n");
		flash_initiate_operation(NVMOP_NOP);
		return ERR_NOT_ERASED;
	}

	/* disable flash write or erase operation */
	writel(NVM_WREN, &nvm_regs_p->ctrl.clr);

	return ERR_OK;
}
```

完整的操作(?)

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

