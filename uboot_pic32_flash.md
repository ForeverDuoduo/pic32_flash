

**以下名字记住咯，忘记了可以回来这里看看**

**flash：闪存**
**u32：unsigned int 32位宽**



```c
#include <common.h>
#include <dm.h>
#include <fdt_support.h>
#include <flash.h>
#include <mach/pic32.h>
#include <wait_bit.h>
```

各类头文件，但是一个都看不懂暂时放着。。



```c
DECLARE_GLOBAL_DATA_PTR;
```

也不知到是什么（



```c
struct pic32_reg_nvm {
	struct pic32_reg_atomic ctrl;
	struct pic32_reg_atomic key;
	struct pic32_reg_atomic addr;
	struct pic32_reg_atomic data;
};
```

pic32_reg_nvm类，这个对象包括四个参数：ctrl，key，addr，data

> 其中：
> struct pic32_reg_atomic {
> 	u32 raw;
> 	u32 clr;
> 	u32 set;
> 	u32 inv;
> };
>
> 嗯，暂时都不知道是做什么用的



```c
/* NVM operations */
#define NVMOP_NOP		0
#define NVMOP_WORD_WRITE	1
#define NVMOP_PAGE_ERASE	4

/* NVM control bits */
#define NVM_WR				BIT(15)
#define NVM_WREN			BIT(14)
#define NVM_WRERR			BIT(13)
#define NVM_LVDERR		BIT(12)

/* NVM programming unlock register */
#define LOCK_KEY			0x0
#define UNLOCK_KEY1		0xaa996655
#define UNLOCK_KEY2		0x556699aa
```

控制信号的宏定义，说明标志位值

#define NVMOP_NOP		0				无操作
#define NVMOP_WORD_WRITE	1	字写入
#define NVMOP_PAGE_ERASE	4	篇擦除

#define NVM_WR			BIT(15)		1111
#define NVM_WREN		BIT(14)		1110
#define NVM_WRERR		BIT(13)	 1101
#define NVM_LVDERR		BIT(12)	1100
分别指示不同工作模式

#define LOCK_KEY		0x0							锁定钥匙值0
#define UNLOCK_KEY1		0xaa996655		解锁钥匙1   1010 1001 0110 0101
#define UNLOCK_KEY2		0x556699aa		解锁钥匙2   0101 0110 1001 1010

```c
flash_info_t flash_info[CONFIG_SYS_MAX_FLASH_BANKS];
```

不知道干嘛用的（



```c
static struct pic32_reg_nvm *nvm_regs_p;
```

​	静态结构体对象 nvm_regs_p



```c
static inline void flash_initiate_operation(u32 nvmop)
{
	/* set operation */
	writel(nvmop, &nvm_regs_p->ctrl.raw);

	/* enable flash write */
	writel(NVM_WREN, &nvm_regs_p->ctrl.set);

	/* unlock sequence */
	writel(LOCK_KEY, &nvm_regs_p->key.raw);
	writel(UNLOCK_KEY1, &nvm_regs_p->key.raw);
	writel(UNLOCK_KEY2, &nvm_regs_p->key.raw);

	/* initiate operation */
	writel(NVM_WR, &nvm_regs_p->ctrl.set);
}
```

flash_initiate_operation()

初始化flash参数，包括：
	set operation 设置参数
	enable flash write 使能
	unlock sequence 解锁
	initiate operation 初始化

* inline关键字是C99标准的型关键字，其作用是将函数展开，把函数的代码复制到每一个调用处。这样调用函数的过程就可以直接执行函数代码，而不发生跳转、压栈等一般性函数操作。可以节省时间，也会提高程序的执行速度。



```c
static int flash_wait_till_busy(const char *func, ulong timeout)
{
	int ret = wait_for_bit(__func__, &nvm_regs_p->ctrl.raw,
			       NVM_WR, false, timeout, false);

	return ret ? ERR_TIMOUT : ERR_OK;
}
```

flash_wait_till_busy

询问flash是否繁忙，超时返回ERR_TIMOUT，没有就会返回ERR_OK

> 其中的wait_for_bit：
>
> ```c
> static inline int wait_for_bit(const char *prefix, const u32 *reg,
> 			       const u32 mask, const bool set,
> 			       const unsigned int timeout_ms,
> 			       const bool breakable)
> {
> 	u32 val;
> 	unsigned long start = get_timer(0);
> 
> 	while (1) {
> 		val = readl(reg);
> 
> 		if (!set)
> 			val = ~val;
> 
> 		if ((val & mask) == mask)
> 			return 0;
> 
> 		if (get_timer(start) > timeout_ms)
> 			break;
> 
> 		if (breakable && ctrlc()) {
> 			puts("Abort\n");
> 			return -EINTR;
> 		}
> 
> 		udelay(1);
> 		WATCHDOG_RESET();
> 	}
> 
> ```





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

flash_complete_operation

完整的操作？

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



```c
/*
 * Erase flash sectors, returns:
 * ERR_OK - OK
 * ERR_INVAL - invalid sector arguments
 * ERR_TIMOUT - write timeout
 * ERR_NOT_ERASED - Flash not erased
 * ERR_UNKNOWN_FLASH_VENDOR - incorrect flash
 */
```

说明各个常量名的含义

> ERR_OK：有效
>
> ERR_INVAL：无效扇区参数
>
> ERR_TIMOUT：等待超时
>
> ERR_NOT_ERASED：存储区域没有被擦除
>
> ERR_UNKNOWN_FLASH_VENDOR：不正确的闪存信息



```c
int write_buff(flash_info_t *info, uchar *src, ulong addr, ulong cnt)
{
	ulong dst, tmp_le, len = cnt;
	int i, l, rc;
	uchar *cp;

	/* get lower word aligned address */
	dst = (addr & ~3);

	/* handle unaligned start bytes */
	l = addr - dst;
	if (l != 0) {
		tmp_le = 0;
		for (i = 0, cp = (uchar *)dst; i < l; ++i, ++cp)
			tmp_le |= *cp << (i * 8);

		for (; (i < 4) && (cnt > 0); ++i, ++src, --cnt, ++cp)
			tmp_le |= *src << (i * 8);

		for (; (cnt == 0) && (i < 4); ++i, ++cp)
			tmp_le |= *cp << (i * 8);

		rc = write_word(info, dst, tmp_le);
		if (rc)
			goto out;

		dst += 4;
	}

	/* handle word aligned part */
	while (cnt >= 4) {
		tmp_le = src[0] | src[1] << 8 | src[2] << 16 | src[3] << 24;
		rc = write_word(info, dst, tmp_le);
		if (rc)
			goto out;
		src += 4;
		dst += 4;
		cnt -= 4;
	}

	if (cnt == 0) {
		rc = ERR_OK;
		goto out;
	}

	/* handle unaligned tail bytes */
	tmp_le = 0;
	for (i = 0, cp = (uchar *)dst; (i < 4) && (cnt > 0); ++i, ++cp) {
		tmp_le |= *src++ << (i * 8);
		--cnt;
	}

	for (; i < 4; ++i, ++cp)
		tmp_le |= *cp << (i * 8);

	rc = write_word(info, dst, tmp_le);
out:
	/*
	 * flash content updated by nvm controller but CPU cache might
	 * have stale data, so invalidate dcache.
	 */
	invalidate_dcache_range(addr, addr + len);

	printf(" done\n");
	return rc;
}
```

write_buff：

向闪存内写入数据

> 第一步：处理非字节对齐开始的数据
>
> 第二步：处理字对齐数据
>
> 第三步：处理非字节对齐的结尾数据

细节还没写



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

	/* calc total flash size */
	for (bank = 0; bank < CONFIG_SYS_MAX_FLASH_BANKS; ++bank)
		size += flash_info[bank].size;

	return size;
}
```



```c

static void pic32_flash_bank_init(flash_info_t *info,
				  ulong base, ulong size)
{
	ulong sect_size;
	int sect;

	/* device & manufacturer code */
	info->flash_id = FLASH_MAN_MCHP | FLASH_MCHP100T;
	info->sector_count = CONFIG_SYS_MAX_FLASH_SECT;
	info->size = size;

	/* update sector (i.e page) info */
	sect_size = info->size / info->sector_count;
	for (sect = 0; sect < info->sector_count; sect++) {
		info->start[sect] = base;
		/* protect each sector by default */
		info->protect[sect] = 1;
		base += sect_size;
	}
}
```



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



```c
static const struct udevice_id pic32_flash_ids[] = {
	{ .compatible = "microchip,pic32mzda-flash" },
	{}
};

U_BOOT_DRIVER(pic32_flash) = {
	.name	= "pic32_flash",
	.id	= UCLASS_MTD,
	.of_match = pic32_flash_ids,
	.probe	= pic32_flash_probe,
};
```





