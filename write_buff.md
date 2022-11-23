# write_buff

```c
int write_buff(flash_info_t *info, uchar *src, ulong addr, ulong cnt)
{
	ulong dst, tmp_le, len = cnt;
	int i, l, rc;
	uchar *cp;

	/* get lower word aligned address */
	dst = (addr & ~3);//dst为地址addr去掉低3位，或者说8字节未对齐的部分

	/* handle unaligned start bytes */
	l = addr - dst;//数据段首如果有没8字节对齐的数据，则处理这部分
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
	while (cnt >= 4) {//处理字节对齐的部分
		tmp_le = src[0] | src[1] << 8 | src[2] << 16 | src[3] << 24;
		rc = write_word(info, dst, tmp_le);//调用字写入函数
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
	tmp_le = 0;//再处理数据段尾未8字节对齐的数据
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
/*
 * Copy memory to flash, returns:
 * ERR_OK - OK
 * ERR_TIMOUT - write timeout
 * ERR_NOT_ERASED - Flash not erased
 */
```

write_buff：

向闪存内写入数据

> 第一步：处理非字节对齐开始的数据
>
> 第二步：处理字对齐数据
>
> 第三步：处理非字节对齐的结尾数据

> ```c
> 	dst = (addr & ~3);
> 
> 	l = addr - dst;
> ```
>
> dst为地址addr去掉低3位，或者说8字节
>
> 如果addr是8字节对齐，l就等于0，否则不为0，需要处理非8字节对齐的数据
>
> 处理非8字节对齐：
>
> ```c
> if (l != 0) {
> 		tmp_le = 0;
> 		for (i = 0, cp = (uchar *)dst; i < l; ++i, ++cp)
> 			tmp_le |= *cp << (i * 8);
> 
> 		for (; (i < 4) && (cnt > 0); ++i, ++src, --cnt, ++cp)
> 			tmp_le |= *src << (i * 8);
> 
> 		for (; (cnt == 0) && (i < 4); ++i, ++cp)
> 			tmp_le |= *cp << (i * 8);
> 
> 		rc = write_word(info, dst, tmp_le);
> 		if (rc)
> 			goto out;
> 
> 		dst += 4;
> 	}
> ```
>
> 处理已经字节对齐的数据：
>
> ```c
> while (cnt >= 4) {
> 		tmp_le = src[0] | src[1] << 8 | src[2] << 16 | src[3] << 24;
> 		rc = write_word(info, dst, tmp_le);
> 		if (rc)
> 			goto out;
> 		src += 4;
> 		dst += 4;
> 		cnt -= 4;
> 	}
> 
> 	if (cnt == 0) {
> 		rc = ERR_OK;
> 		goto out;
> 	}
> ```
>
> 处理尾巴没有对齐的数据：
>
> ```c
> tmp_le = 0;
> 	for (i = 0, cp = (uchar *)dst; (i < 4) && (cnt > 0); ++i, ++cp) {
> 		tmp_le |= *src++ << (i * 8);
> 		--cnt;
> 	}
> 
> 	for (; i < 4; ++i, ++cp)
> 		tmp_le |= *cp << (i * 8);
> ```
>
> 

