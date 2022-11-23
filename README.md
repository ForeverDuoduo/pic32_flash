# pic32_flash

仅仅只是按照自己的想法分析了一下代码

~~很抱歉有很多翻译烂的地方，我逝英语苦手~~

**flash：闪存**

**u32：unsigned int 32位宽**



##### 说明各个常量名的含义

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
/*-----------------------------------------------------------------------
 * return codes from flash_write():
 */
#define ERR_OK				0
#define ERR_TIMOUT			1
#define ERR_NOT_ERASED			2
#define ERR_PROTECTED			4
#define ERR_INVAL			8
#define ERR_ALIGN			16
#define ERR_UNKNOWN_FLASH_VENDOR	32
#define ERR_UNKNOWN_FLASH_TYPE		64
#define ERR_PROG_ERROR			128
#define ERR_ABORTED			256
```



> | 函数名                   | 名                     | 状态 |
> | ------------------------ | ---------------------- | ---- |
> | flash_initiate_operation | 寄存器解锁             | √    |
> | flash_wait_till_busy     |                        |      |
> | flash_complete_operation | 控制寄存器             | √    |
> | flash_erase              | 程序闪存擦除           | √    |
> | page_erase               |                        |      |
> | write_word               | 向闪存内写入一字节数据 | √    |
> | write_buff               | 向闪存内写入一段数据   | √    |
> | flash_print_info         |                        | √    |
> | flash_init               |                        | √    |
> | pic32_flash_bank_init    |                        | √    |
> | pic32_flash_probe        |                        | √    |
> | pic32_flash_ids          |                        |      |
> | pic32_flash              |                        |      |
