# flash_initiate_operation

```c
static inline void flash_initiate_operation(u32 nvmop)
{
	/* set operation */
	writel(nvmop, &nvm_regs_p->ctrl.raw);//NVMOP<3:0>:NVM 操作位

	/* enable flash write */
	writel(NVM_WREN, &nvm_regs_p->ctrl.set);//WREN:写使能位

	/* unlock sequence */
	writel(LOCK_KEY, &nvm_regs_p->key.raw);
	writel(UNLOCK_KEY1, &nvm_regs_p->key.raw);
	writel(UNLOCK_KEY2, &nvm_regs_p->key.raw);
	//将解锁序列依次写入nvm_regs_p->key

	/* initiate operation */
	writel(NVM_WR, &nvm_regs_p->ctrl.set);
}
```

启动 NVM 操作 (解锁序列示例)

>   一旦解锁代码已写入 NVMKEY 寄存器，与闪存控制器在同一外设总线的下一个活动 将复位锁定。最终，仅可以使用原子操作。使用结构设置寄存器位域会编译为读 - 修 改 - 写操作，导致失败。

> ```
> NVMOP<3:0>:NVM 操作位
> 仅当 WREN = 0 时，这些位才可写。
> ```

> ```c
> /* NVM programming unlock register */
> #define LOCK_KEY			0x0
> #define UNLOCK_KEY1		0xaa996655
> #define UNLOCK_KEY2		0x556699aa
> ```
>
> 解锁序列



* inline关键字是C99标准的型关键字，其作用是将函数展开，把函数的代码复制到每一个调用处。这样调用函数的过程就可以直接执行函数代码，而不发生跳转、压栈等一般性函数操作。可以节省时间，也会提高程序的执行速度。