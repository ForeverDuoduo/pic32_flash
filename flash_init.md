# flash_init

```c
unsigned long flash_init(void)
{
	unsigned long size = 0;
	struct udevice *dev;
	int bank;

	/* probe every MTD device */
	for (uclass_first_device(UCLASS_MTD, &dev); dev;
	     uclass_next_device(&dev)) {
		/* nop */
	}

	/* calc total flash size */ //计算所有闪存的大小和并作为结果返回
	for (bank = 0; bank < CONFIG_SYS_MAX_FLASH_BANKS; ++bank)
		size += flash_info[bank].size;

	return size;
}
```





> ```c
> /* probe every MTD device */
> 	for (uclass_first_device(UCLASS_MTD, &dev); dev;
> 	     uclass_next_device(&dev)) {
> 		/* nop */
> 	}
> ```
>
> 检测每一个device是否有问题
>
> （虽然循环体nop操作了。。但应该出错了会报错）



> ```c
> 	/* calc total flash size */
> 	for (bank = 0; bank < CONFIG_SYS_MAX_FLASH_BANKS; ++bank)
> 		size += flash_info[bank].size;
> ```
>
> #define CONFIG_SYS_MAX_FLASH_BANKS	1/2
>
> flash_info_t flash_info[CONFIG_SYS_MAX_FLASH_BANKS];
>
> 一般是1到2个bank
>
> size累加，也就是计算全部的闪存大小



> ```c
> int uclass_first_device(enum uclass_id id, struct udevice **devp);
> 
> /**
>  * uclass_first_device_err() - Get the first device in a uclass
>  *
>  * The device returned is probed if necessary, and ready for use
>  *
>  * @id: Uclass ID to look up
>  * @devp: Returns pointer to the first device in that uclass, or NULL if none
>  * @return 0 if found, -ENODEV if not found, other -ve on error
>  */
> int uclass_first_device_err(enum uclass_id id, struct udevice **devp);
> 
> /**
>  * uclass_next_device() - Get the next device in a uclass
>  *
>  * The device returned is probed if necessary, and ready for use
>  *
>  * This function is useful to start iterating through a list of devices which
>  * are functioning correctly and can be probed.
>  *
>  * @devp: On entry, pointer to device to lookup. On exit, returns pointer
>  * to the next device in the uclass if no error occurred, or NULL if there is
>  * no next device, or an error occurred with that next device.
>  * @return 0 if OK (found or not found), other -ve on error
>  */
> int uclass_next_device(struct udevice **devp);
> 
> /**
>  * uclass_first_device() - Get the first device in a uclass
>  *
>  * The device returned is probed if necessary, and ready for use
>  *
>  * This function is useful to start iterating through a list of devices which
>  * are functioning correctly and can be probed.
>  *
>  * @id: Uclass ID to look up
>  * @devp: Returns pointer to the first device in that uclass, or NULL if there
>  * is no first device
>  * @return 0 if OK (found or not found), other -ve on error. If an error occurs
>  * it is still possible to move to the next device.
>  */
> ```
>
> 

