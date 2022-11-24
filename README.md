# pic32_flash

仅仅只是按照自己的想法分析了一下代码

~~很抱歉有很多翻译烂的地方，我逝英语苦手~~

**flash：闪存**

**NAND：与非门电路**

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
#define ERR_OK			没有错误
#define ERR_TIMOUT		超时
#define ERR_NOT_ERASED	擦除错误
#define ERR_PROTECTED	被保护
#define ERR_INVAL		无效
#define ERR_ALIGN		对齐错误
#define ERR_UNKNOWN_FLASH_VENDOR	接触屏蔽
#define ERR_UNKNOWN_FLASH_TYPE		类型错误
#define ERR_PROG_ERROR		掠夺错误
#define ERR_ABORTED			中止错误
```

>   1.没有错误 2.超时错误 3.没有擦除错误 4.被保护的错误 5.无效错误 6.错误对齐 7.接触屏蔽 8. 错误类型 9.掠夺错误 10.中止错误

```c
/* NVM operations */
#define NVMOP_NOP		0
#define NVMOP_WORD_WRITE	1
#define NVMOP_PAGE_ERASE	4

/* NVM control bits */
#define NVM_WR			BIT(15)
#define NVM_WREN		BIT(14)
#define NVM_WRERR		BIT(13)
#define NVM_LVDERR		BIT(12)

/* NVM programming unlock register */
#define LOCK_KEY		0x0
#define UNLOCK_KEY1		0xaa996655
#define UNLOCK_KEY2		0x556699aa

```







> | 函数名                   | 名                                      | 状态 |
> | ------------------------ | --------------------------------------- | ---- |
> | flash_initiate_operation | 寄存器解锁/启动 NVM 操作 (解锁序列示例) | √    |
> | flash_wait_till_busy     | 询问是否忙/超时                         | √    |
> | flash_complete_operation | 控制寄存器                              | √    |
> | flash_erase              | 程序闪存擦除                            | √    |
> | page_erase               | 页擦除（但是return 0）                  | √    |
> | write_word               | 向闪存内写入一字节数据/字编程           | √    |
> | write_buff               | 向闪存内写入一段数据/4字编程            | √    |
> | flash_print_info         | 打印设备信息                            | √    |
> | flash_init               | 计算所有闪存大小和                      | √    |
> | pic32_flash_bank_init    | 闪存存储区初始化                        | √    |
> | pic32_flash_probe        |                                         | √    |
> | U_BOOT_DRIVER            | 驱动初始化                              | √    |
> | pic32_flash              |                                         |      |
