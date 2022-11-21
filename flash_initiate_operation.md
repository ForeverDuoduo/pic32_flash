# flash_initiate_operation

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

初始化flash参数，包括：
	set operation 设置参数
	enable flash write 使能
	unlock sequence 解锁
	initiate operation 初始化

* inline关键字是C99标准的型关键字，其作用是将函数展开，把函数的代码复制到每一个调用处。这样调用函数的过程就可以直接执行函数代码，而不发生跳转、压栈等一般性函数操作。可以节省时间，也会提高程序的执行速度。

* writel原型：

  \#include 

  void writel (unsigned char data , unsigned short addr )