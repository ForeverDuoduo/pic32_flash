# flash_print_info

```c
void flash_print_info(flash_info_t *info)
{
	int i;

	if (info->flash_id == FLASH_UNKNOWN) {
		printf("missing or unknown FLASH type\n");
		return;
	}

	switch (info->flash_id & FLASH_VENDMASK) {
	case FLASH_MAN_MCHP:
		printf("Microchip Technology ");
		break;
	default:
		printf("Unknown Vendor ");
		break;
	}

	switch (info->flash_id & FLASH_TYPEMASK) {
	case FLASH_MCHP100T:
		printf("Internal (8 Mbit, 64 x 16k)\n");
		break;
	default:
		printf("Unknown Chip Type\n");
		break;
	}

	printf("  Size: %ld MB in %d Sectors\n",
	       info->size >> 20, info->sector_count);

	printf("  Sector Start Addresses:");
	for (i = 0; i < info->sector_count; ++i) {
		if ((i % 5) == 0)
			printf("\n   ");

		printf(" %08lX%s", info->start[i],
		       info->protect[i] ? " (RO)" : "     ");
	}
	printf("\n");
}
```

flash_print_info：

第一步：if查看info->flash_id，也就是看看是不是一个闪存，不是就退出，是就继续

> #define FLASH_TYPEMASK	0x0000FFFF	/* extract FLASH type	information	\*/
> 掩码 取info->flash_id低16位
>
> #define FLASH_VENDMASK	0xFFFF0000	/* extract FLASH vendor information	*/
> 掩码 取info->flash_id高16位

第二步：看info->flash_id & FLASH_VENDMASK 判断是不是一个Microchip Technology芯片

> #define FLASH_MAN_MCHP	0x02000000	/* Microchip Technology		*/

第三步：看info->flash_id & FLASH_TYPEMASK 判断芯片类型

> #define FLASH_MCHP100T	0x0060		/* MCHP internal (1M = 64K x 16) \*/
> #define FLASH_MCHP100B	0x0061		/* MCHP internal (1M = 64K x 16) */

第四步：输出存储空间大小

> printf("  Size: %ld MB in %d Sectors\n",
> 	info->size >> 20, info->sector_count);

第五步：告知Sector的起始地址