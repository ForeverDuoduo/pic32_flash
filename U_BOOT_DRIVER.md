# U_BOOT_DRIVER

```c
static const struct udevice_id pic32_flash_ids[] = {
	{ .compatible = "microchip,pic32mzda-flash" },		// 对芯片的id号进行兼容 
	{}
};

U_BOOT_DRIVER(pic32_flash) = {			//驱动程序属性
	.name	= "pic32_flash",
	.id	= UCLASS_MTD,
	.of_match = pic32_flash_ids,		//匹配id号 
	.probe	= pic32_flash_probe,		//探针 
};
```

