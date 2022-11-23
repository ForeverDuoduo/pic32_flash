# flash_wait_till_busy

```c
static int flash_wait_till_busy(const char *func, ulong timeout)
{
   int ret = wait_for_bit(__func__, &nvm_regs_p->ctrl.raw,
                NVM_WR, false, timeout, false);

   return ret ? ERR_TIMOUT : ERR_OK;//等待，如果超时会返回ERR_TIMOUT
}
```