# pic32_flash_probe

```c
static int pic32_flash_probe(struct udevice *dev)
{
	void *blob = (void *)gd->fdt_blob;
	int node = dev_of_offset(dev);
	const char *list, *end;
	const fdt32_t *cell;
	unsigned long addr, size;
	int parent, addrc, sizec;
	flash_info_t *info;
	int len, idx;

	/*
	 * decode regs. there are multiple reg tuples, and they need to
	 * match with reg-names.
	 */
	parent = fdt_parent_offset(blob, node);
	fdt_support_default_count_cells(blob, parent, &addrc, &sizec);
	list = fdt_getprop(blob, node, "reg-names", &len);
	if (!list)
		return -ENOENT;

	end = list + len;
	cell = fdt_getprop(blob, node, "reg", &len);
	if (!cell)
		return -ENOENT;

	for (idx = 0, info = &flash_info[0]; list < end;) {
		addr = fdt_translate_address((void *)blob, node, cell + idx);
		size = fdt_addr_to_cpu(cell[idx + addrc]);
		len = strlen(list);
		if (!strncmp(list, "nvm", len)) {
			/* NVM controller */
			nvm_regs_p = ioremap(addr, size);
		} else if (!strncmp(list, "bank", 4)) {
			/* Flash bank: use kseg0 cached address */
			pic32_flash_bank_init(info, CKSEG0ADDR(addr), size);
			info++;
		}
		idx += addrc + sizec;
		list += len + 1;
	}

	/* disable flash write/erase operations */
	writel(NVM_WREN, &nvm_regs_p->ctrl.clr);

#if (CONFIG_SYS_MONITOR_BASE >= CONFIG_SYS_FLASH_BASE)
	/* monitor protection ON by default */
	flash_protect(FLAG_PROTECT_SET,
		      CONFIG_SYS_MONITOR_BASE,
		      CONFIG_SYS_MONITOR_BASE + monitor_flash_len - 1,
		      &flash_info[0]);
#endif

#ifdef CONFIG_ENV_IS_IN_FLASH
	/* ENV protection ON by default */
	flash_protect(FLAG_PROTECT_SET,
		      CONFIG_ENV_ADDR,
		      CONFIG_ENV_ADDR + CONFIG_ENV_SECT_SIZE - 1,
		      &flash_info[0]);
#endif 
	return 0;
}
```







> ```c
> 	for (idx = 0, info = &flash_info[0]; list < end;) {
> 		addr = fdt_translate_address((void *)blob, node, cell + idx);
> 		size = fdt_addr_to_cpu(cell[idx + addrc]);
> 		len = strlen(list);
> 		if (!strncmp(list, "nvm", len)) {
> 			/* NVM controller */
> 			nvm_regs_p = ioremap(addr, size);
> 		} else if (!strncmp(list, "bank", 4)) {
> 			/* Flash bank: use kseg0 cached address */
> 			pic32_flash_bank_init(info, CKSEG0ADDR(addr), size);
> 			info++;
> 		}
> 		idx += addrc + sizec;
> 		list += len + 1;
> 	}
> ```
>
> 
>
> #define CKSEG0			\_CONST64_(0xffffffff80000000)
>
> #define CKSEG0ADDR(a)		(CPHYSADDR(a) | CKSEG0)

