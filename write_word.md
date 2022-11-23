# write_word

```c
/* Write a word to flash */
static int write_word(flash_info_t *info, ulong dest, ulong word)
{
	ulong flags;
	int rc;

	/* read flash to check if it is sufficiently erased */
	if ((readl((void __iomem *)dest) & word) != word) {
		printf("Error, Flash not erased!\n");
		return ERR_NOT_ERASED;
	}//先检查falsh是否被擦除，擦除失败返回错误没有擦除

	/* disable interrupts */
	flags = disable_interrupts();//不允许中断

	/* update destination page address (physical) */
	writel(CPHYSADDR(dest), &nvm_regs_p->addr.raw);
	writel(word, &nvm_regs_p->data.raw);//物理层面上更新目的页

	/* word write */
	flash_initiate_operation(NVMOP_WORD_WRITE);//更新numop为word write类型

	/* wait for operation to complete */
	rc = flash_wait_till_busy(__func__, CONFIG_SYS_FLASH_WRITE_TOUT);//如果忙则等待

	/* re-enable interrupts if necessary */
	if (flags)
		enable_interrupts();//如果上面执行了不允许中断则这里恢复

	if (rc != ERR_OK)
		return rc;

	return flash_complete_operation();
}

```

>   ```c
>   int disable_interrupts(void)
>   {
>   	int status = read_aux_reg(ARC_AUX_STATUS32);
>   	int state = (status & (E1_MASK | E2_MASK)) ? 1 : 0;
>   
>   	status &= ~(E1_MASK | E2_MASK);
>   	/* STATUS32 register is updated indirectly with "FLAG" instruction */
>   	__asm__("flag %0" : : "r" (status));
>   	return state;
>   }//关闭中断
>   
>   void enable_interrupts(void)
>   {
>   	unsigned int status = read_aux_reg(ARC_AUX_STATUS32);
>   
>   	status |= E1_MASK | E2_MASK;
>   	/* STATUS32 register is updated indirectly with "FLAG" instruction */
>   	__asm__("flag %0" : : "r" (status));
>   }//启动中断
>   ```
>
>   