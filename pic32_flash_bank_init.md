# pic32_flash_bank_init

```c
static void pic32_flash_bank_init(flash_info_t *info,
				  ulong base, ulong size)
{
	ulong sect_size;
	int sect;

	/* device & manufacturer code */ //根据芯片型号和厂商信息修改ID号
	info->flash_id = FLASH_MAN_MCHP | FLASH_MCHP100T;
	info->sector_count = CONFIG_SYS_MAX_FLASH_SECT;
	info->size = size;

	/* update sector (i.e page) info */ //更新扇区信息
	sect_size = info->size / info->sector_count;//sect_size为共有多少个扇区
	for (sect = 0; sect < info->sector_count; sect++) {
		info->start[sect] = base;//添加每个扇区的起始地址信息
		/* protect each sector by default */
		info->protect[sect] = 1;
		base += sect_size;//更新下一个扇区的地址
	}
}
```







初始化flash_bank入参为flash_info_t *info闪存信息，ulong base，ulong size容量

> #define FLASH_MAN_MCHP	0x02000000	/* Microchip Technology		*/
>
> #define FLASH_MCHP100T	0x0060		/* MCHP internal (1M = 64K x 16) */



