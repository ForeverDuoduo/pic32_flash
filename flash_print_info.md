# flash_print_info

```c
void flash_print_info(flash_info_t *info)
{
	int i;

	if (info->flash_id == FLASH_UNKNOWN) {//检查是否是未知类型的参数
		printf("missing or unknown FLASH type\n");//识别失败打印警告并返回
		return;
	}

	switch (info->flash_id & FLASH_VENDMASK) {//打印芯片的出厂公司
	case FLASH_MAN_MCHP:
		printf("Microchip Technology ");
		break;
	default:
		printf("Unknown Vendor ");
		break;
	}

	switch (info->flash_id & FLASH_TYPEMASK) {//打印闪存的型号
	case FLASH_MCHP100T:
		printf("Internal (8 Mbit, 64 x 16k)\n");
		break;
	default:
		printf("Unknown Chip Type\n");
		break;
	}

	printf("  Size: %ld MB in %d Sectors\n",//打印总存储容量
	       info->size >> 20, info->sector_count);

	printf("  Sector Start Addresses:");//打印各闪存的起始地址
	for (i = 0; i < info->sector_count; ++i) {
		if ((i % 5) == 0)
			printf("\n   ");

		printf(" %08lX%s", info->start[i],
		       info->protect[i] ? " (RO)" : "     ");
	}
	printf("\n");//结束
}
```

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

第五步：告知每个扇区的起始地址