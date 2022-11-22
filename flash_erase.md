# flash_erase

```c
int flash_erase(flash_info_t *info, int s_first, int s_last)
{
	ulong sect_start, sect_end, flags;
	int prot, sect;
	int rc;

	if ((info->flash_id & FLASH_VENDMASK) != FLASH_MAN_MCHP) {		
		printf("Can't erase unknown flash type %08lx - aborted\n",  // 无法擦除未知flash类型 
		       info->flash_id);
		return ERR_UNKNOWN_FLASH_VENDOR;	//ERR_UNKNOWN_FLASH_VENDOR：未知flash类型
	}

	if ((s_first < 0) || (s_first > s_last)) {
		printf("- no sectors to erase\n");		//没有需要擦除的扇区 
		return ERR_INVAL;						//ERR_INVAL：无效扇区擦除参数
	}

	prot = 0;
	for (sect = s_first; sect <= s_last; ++sect) {
		if (info->protect[sect])
			prot++;
	}

	if (prot)						//如果被保护则输出有多少个扇区被保护 
		printf("- Warning: %d protected sectors will not be erased!\n",
		       prot);
	else
		printf("\n");			//没有被保护的扇区这个问题

	/* erase on unprotected sectors */
	for (sect = s_first; sect <= s_last; sect++) {//将未被保护的扇区擦除 
		if (info->protect[sect])
			continue;

		/* disable interrupts */  //禁用中断 
		flags = disable_interrupts();

		/* write destination page address (physical) */  //写入目标页的物理地址 
		sect_start = CPHYSADDR(info->start[sect]);
		writel(sect_start, &nvm_regs_p->addr.raw);

		/* page erase */			//开始擦除 
		flash_initiate_operation(NVMOP_PAGE_ERASE);

		/* wait */
		rc = flash_wait_till_busy(__func__,
					  CONFIG_SYS_FLASH_ERASE_TOUT);

		/* re-enable interrupts if necessary */  //判断是否需要启用中断 
		if (flags)
			enable_interrupts();

		if (rc != ERR_OK)
			return rc;

		rc = flash_complete_operation();
		if (rc != ERR_OK)
			return rc;

		/*
		 * flash content is updated but cache might contain stale//flash已跟新，但缓存过时可能 
		 * data, so invalidate dcache.   //因此缓存失效 
		 */
		sect_end = info->start[sect] + info->size / info->sector_count;
		invalidate_dcache_range(info->start[sect], sect_end);
	}

	printf(" done\n");
	return ERR_OK;
}
```

