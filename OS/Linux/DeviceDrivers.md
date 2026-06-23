# Linux Device Drivers — MAANG Interview Questions
> 10 Years Embedded Experience | Basic → Intermediate → Advanced → Expert
> Format: Question → Counter Questions (Interviewer probes deeper)
> ⚠️ No answers — test yourself

---

## BASIC LEVEL

### 1. What is a device driver?
- Counter: Why can't application code directly access hardware?
- Counter: What is the difference between driver and firmware?
- Counter: Can a driver run in user space? Give an example.
- Counter: What happens if a driver crashes?

---

### 2. What is the difference between character, block, and network device drivers?
- Counter: When would you choose character over block?
- Counter: Is a USB storage device a character or block device — explain internally?
- Counter: Can one physical device have both character and block interfaces?
- Counter: What does `/dev/sda` vs `/dev/sda1` tell you architecturally?

---

### 3. What is a kernel module?
- Counter: What is the difference between `insmod` and `modprobe`?
- Counter: What happens at the assembly level when you `insmod` a module?
- Counter: Can a module be loaded multiple times?
- Counter: What is module versioning and why does it matter?
- Counter: What happens if a module is loaded and the kernel version mismatches?

---

### 4. What are `init_module` and `cleanup_module`?
- Counter: What is `module_init()` and `module_exit()` macro and how do they differ?
- Counter: What happens if `cleanup_module` is not called properly?
- Counter: Can `module_init` fail? What happens then?
- Counter: What is the order of initialization if multiple modules depend on each other?

---

### 5. What is `file_operations` structure?
- Counter: Walk me through every field in `file_operations`.
- Counter: What is `unlocked_ioctl` vs `ioctl` — why was it changed?
- Counter: What is `llseek` — when would a char device need it?
- Counter: What does returning `-EINVAL` from `open()` mean to the caller?
- Counter: What is `fasync` in `file_operations`?

---

### 6. How do you create a device node?
- Counter: What is major and minor number — how are they used internally?
- Counter: What is `alloc_chrdev_region` vs `register_chrdev_region`?
- Counter: How does udev know to create `/dev/mydevice`?
- Counter: What is the difference between static and dynamic major number allocation?

---

### 7. What is `ioctl`?
- Counter: Why not just use `read`/`write` instead of `ioctl`?
- Counter: How do you define an ioctl number? What is `_IO`, `_IOR`, `_IOW`, `_IOWR`?
- Counter: What is `compat_ioctl` and when is it needed?
- Counter: How do you pass complex structures through ioctl safely?

---

### 8. What is `copy_to_user` and `copy_from_user`?
- Counter: Why can't you just use `memcpy`?
- Counter: What happens if the user pointer is NULL or invalid?
- Counter: What is the return value of `copy_to_user` and what does it mean?
- Counter: What is `__user` annotation — what does the compiler/checker do with it?
- Counter: Can `copy_from_user` sleep?

---

## INTERMEDIATE LEVEL

### 9. What is interrupt handling in a Linux driver?
- Counter: What is top half vs bottom half — explain with real use case.
- Counter: What context restrictions apply in an ISR?
- Counter: What is `irqreturn_t` — what are valid return values?
- Counter: What is `IRQF_SHARED` — explain with example.
- Counter: What happens if you call `kmalloc` inside an ISR?

---

### 10. What is tasklet vs workqueue — when do you choose each?
- Counter: Can a tasklet sleep? Why?
- Counter: Can a workqueue sleep? Why?
- Counter: What is `DECLARE_TASKLET` vs `tasklet_init`?
- Counter: What is a threaded IRQ — how is it different from workqueue?
- Counter: What is `request_threaded_irq` and when to use it?

---

### 11. What is DMA in a Linux driver?
- Counter: What is coherent vs streaming DMA?
- Counter: What is `dma_alloc_coherent` and what does it guarantee?
- Counter: Why do you need `dma_map_single` before DMA transfer?
- Counter: What is IOMMU — how does it affect DMA?
- Counter: What is DMA direction (`DMA_TO_DEVICE`, `DMA_FROM_DEVICE`) — why does it matter?
- Counter: What happens if you forget to call `dma_unmap_single`?

---

### 12. What is platform driver and platform device?
- Counter: What is the difference between platform bus and PCI bus?
- Counter: How does platform driver `probe` get called?
- Counter: What is `platform_get_resource` — what resources can it retrieve?
- Counter: What is the role of device tree in platform driver binding?
- Counter: What is `of_match_table` and `compatible` string matching?

---

### 13. What is `probe` and `remove` in a driver?
- Counter: At what point exactly is `probe` called?
- Counter: What should you do in `probe` vs what should you do in `open`?
- Counter: What happens if `probe` returns an error?
- Counter: What is deferred probe — why does it exist?
- Counter: What is `probe_type` = `PROBE_PREFER_ASYNCHRONOUS`?

---

### 14. What is device tree binding for a driver?
- Counter: How does `of_device_id` table work?
- Counter: What is `of_get_named_gpio` — where does the GPIO come from?
- Counter: How do you get clock from device tree inside driver?
- Counter: What is `devm_ioremap_resource` vs `ioremap`?
- Counter: How do you parse custom properties from device tree in driver?

---

### 15. What is `devm_` (device managed resources)?
- Counter: What problem does `devm_` solve?
- Counter: What happens to `devm_kmalloc` memory when device is removed?
- Counter: Can you mix `devm_` and manual resource management?
- Counter: What is `devm_add_action` — when would you use it?
- Counter: Are there any downsides to using `devm_` everywhere?

---

### 16. What is regmap in Linux drivers?
- Counter: What problem does regmap solve?
- Counter: How do you initialize regmap for I2C device?
- Counter: What is `regmap_read`, `regmap_write`, `regmap_update_bits`?
- Counter: What is regmap caching — what cache types exist?
- Counter: How do you debug regmap register access?

---

### 17. What is `mmap` in a device driver?
- Counter: Why would a driver want to support `mmap`?
- Counter: What is `vm_ops` and `vm_fault`?
- Counter: What is `remap_pfn_range` vs `vm_insert_page`?
- Counter: What is `pgprot_noncached` — when is it needed?
- Counter: What are security risks of exposing memory via mmap?

---

### 18. What is I2C driver in Linux?
- Counter: What is `i2c_driver` vs `i2c_client`?
- Counter: How does I2C driver bind to I2C device defined in device tree?
- Counter: What is `i2c_transfer` vs `i2c_smbus_read_byte_data`?
- Counter: What is SMBus — how is it related to I2C?
- Counter: What happens if I2C NACK occurs inside driver?

---

### 19. What is SPI driver in Linux?
- Counter: What is `spi_driver` vs `spi_device`?
- Counter: What is `spi_transfer` and `spi_message`?
- Counter: How do you configure SPI mode, speed in device tree?
- Counter: What is `spi_sync` vs `spi_async`?
- Counter: How do you handle SPI chip select in multi-slave scenario?

---

## ADVANCED LEVEL

### 20. What is kernel locking — explain all lock types?
- Counter: When do you use spinlock vs mutex?
- Counter: What is `spin_lock_irqsave` — why do you need to save IRQ flags?
- Counter: What is read-write spinlock — when is it beneficial?
- Counter: What is RCU (Read-Copy-Update) — explain the mechanism?
- Counter: What is seqlock — when would you use it in a driver?
- Counter: What is the ABA problem — can it affect spinlocks?

---

### 21. What is memory barrier in Linux kernel?
- Counter: What is `wmb()`, `rmb()`, `mb()` — when do you need each?
- Counter: Why does compiler reordering matter even on single core?
- Counter: What is `smp_mb()` vs `mb()` — difference?
- Counter: What is `READ_ONCE` and `WRITE_ONCE` — why use them?
- Counter: What is `dma_wmb()` and when is it needed?

---

### 22. What is MMIO (Memory-Mapped I/O) in driver?
- Counter: What is the difference between `ioremap` and `ioremap_nocache`?
- Counter: What is `readl`/`writel` vs direct pointer dereference?
- Counter: What is `ioread32be` — when would you use it?
- Counter: What happens if you access MMIO without `ioremap`?
- Counter: What is `devm_ioremap_resource` and what does it check?

---

### 23. What is scatter-gather DMA?
- Counter: Why is scatter-gather needed — what problem does it solve?
- Counter: What is `scatterlist` and `sg_table`?
- Counter: What is `dma_map_sg` — what does it return?
- Counter: What is ION allocator — where is it used?
- Counter: How do you handle partial DMA completion?

---

### 24. What is kernel memory allocation — all types?
- Counter: What is the difference between `kmalloc`, `vmalloc`, `kzalloc`, `kcalloc`?
- Counter: What is slab allocator — what problem does it solve?
- Counter: What is GFP flags — explain `GFP_KERNEL`, `GFP_ATOMIC`, `GFP_DMA`?
- Counter: What is SLUB vs SLAB vs SLOB?
- Counter: When would `vmalloc` cause issues in a driver compared to `kmalloc`?
- Counter: What is `alloc_pages` — when does a driver need it?

---

### 25. What is power management in a driver?
- Counter: What is `dev_pm_ops` — what callbacks does it have?
- Counter: What is the difference between `suspend` and `runtime_suspend`?
- Counter: What is runtime PM — how do you enable it in driver?
- Counter: What is `pm_runtime_get_sync` vs `pm_runtime_put`?
- Counter: What is wakeup source in PM — how do you register one?
- Counter: What happens if a driver doesn't implement `suspend` properly?

---

### 26. What is GPIO driver in Linux?
- Counter: What is `gpiochip` vs consumer GPIO API?
- Counter: What is `gpio_request` vs `gpiod_get`?
- Counter: What is `gpio_to_irq` — how does it work internally?
- Counter: What is open-drain GPIO — how is it configured?
- Counter: What is GPIO descriptor vs legacy GPIO API — why is legacy deprecated?

---

### 27. What is clock framework in Linux driver?
- Counter: What is `clk_get`, `clk_enable`, `clk_set_rate`?
- Counter: What is CCF (Common Clock Framework)?
- Counter: What is clock gating — why is it important in drivers?
- Counter: How do you register a clock provider?
- Counter: What happens if you call `clk_enable` without `clk_prepare` first?

---

### 28. What is regulator framework in Linux driver?
- Counter: What is voltage regulator vs current regulator in driver context?
- Counter: What is `regulator_get`, `regulator_enable`, `regulator_set_voltage`?
- Counter: What is regulator consumer vs provider driver?
- Counter: What is regulator coupling — when does it matter?
- Counter: What happens if regulator enable is reference counted incorrectly?

---

### 29. What is completion in Linux kernel?
- Counter: What problem does `completion` solve that semaphore doesn't?
- Counter: What is `wait_for_completion_interruptible` vs `wait_for_completion_timeout`?
- Counter: What is `complete_all` vs `complete` — when do you need each?
- Counter: Can you use completion in interrupt context?
- Counter: What is the difference between completion and waitqueue?

---

### 30. What is kobject and sysfs in driver?
- Counter: What is the relationship between `kobject` and device model?
- Counter: How do you create a sysfs attribute in a driver?
- Counter: What is `DEVICE_ATTR_RW` macro — what does it generate?
- Counter: What is `sysfs_notify` — when would a driver use it?
- Counter: What are the thread-safety requirements for sysfs `show`/`store` callbacks?

---

## EXPERT LEVEL

### 31. You wrote a driver and it causes kernel panic — how do you debug?
- Counter: You only have serial console, no JTAG — what's your approach?
- Counter: The panic happens inside an ISR — how does that change your debug strategy?
- Counter: Panic shows `NULL pointer dereference` in `probe` — walk me through all possible causes.
- Counter: How do you use `CONFIG_KASAN` to find memory bugs in driver?
- Counter: You get `BUG: sleeping function called from invalid context` — what caused it and how do you fix?

---

### 32. Your driver has a race condition that only appears under load — how do you find it?
- Counter: What kernel configs would you enable to detect races?
- Counter: What is `CONFIG_LOCKDEP` — what does it detect?
- Counter: What is KCSAN (Kernel Concurrency Sanitizer)?
- Counter: What is `CONFIG_DEBUG_SPINLOCK` — what does it catch?
- Counter: How do you reproduce race conditions deterministically?

---

### 33. Your driver has a memory leak — how do you detect and fix it?
- Counter: What is `kmemleak` — how do you enable and use it?
- Counter: What is `slabinfo` — what does it show?
- Counter: Driver works fine for 24hrs then OOM — what's your systematic approach?
- Counter: What is `kmem_cache_create` leak — harder to detect, why?
- Counter: `devm_` is used everywhere but still a leak occurs — how?

---

### 34. Design a high-speed DMA driver for a custom FPGA peripheral — walk me through every decision
- Counter: How do you handle descriptor rings vs single DMA transactions?
- Counter: What is IOMMU impact on your DMA design?
- Counter: How do you handle partial transfers and error recovery?
- Counter: What is cache coherency strategy for DMA buffers?
- Counter: How do you expose this to user space — `mmap` or `ioctl`? Justify.
- Counter: How do you handle hot-unplug while DMA is in progress?

---

### 35. What is PCIe driver architecture in Linux?
- Counter: What is `pci_driver`, `pci_device_id` table — how does binding work?
- Counter: What is BAR (Base Address Register) — how do you map it?
- Counter: What is MSI vs MSI-X vs legacy interrupt in PCIe driver?
- Counter: What is PCIe ASPM — how does driver interact with it?
- Counter: How do you handle PCIe surprise removal in driver?
- Counter: What is PCIe AER (Advanced Error Reporting) — driver responsibility?

---

### 36. What is USB driver in Linux — explain the full stack?
- Counter: What is URB (USB Request Block)?
- Counter: What is the difference between USB gadget driver and USB host driver?
- Counter: What is `usb_bulk_msg` vs `usb_submit_urb`?
- Counter: How do you handle USB disconnect while transfer is in progress?
- Counter: What is USB composite device — how do you write a driver for it?
- Counter: What is XHCI vs EHCI — driver implications?

---

### 37. What is IOMMU and how does it affect driver design?
- Counter: What problem does IOMMU solve for device drivers?
- Counter: What is DMA mapping vs IOMMU mapping?
- Counter: What is `iommu_domain` and `iommu_map`?
- Counter: What is VFIO — how does IOMMU enable device passthrough?
- Counter: What is `CONFIG_IOMMU_DMA` impact on your driver?

---

### 38. What is CMA (Contiguous Memory Allocator)?
- Counter: Why do some drivers need physically contiguous memory?
- Counter: What is the difference between CMA and DMA coherent pool?
- Counter: How do you configure CMA size for your platform?
- Counter: What is `dma_contiguous_reserve` — when is it called?
- Counter: What are the failure modes of CMA allocation and how to handle them?

---

### 39. You need to write a driver that handles 1 million interrupts per second — what are your concerns?
- Counter: What is interrupt coalescing — how do you implement it?
- Counter: What is NAPI — can it apply to non-network drivers?
- Counter: How do you measure actual interrupt latency and overhead?
- Counter: What is threaded IRQ overhead vs hard IRQ for this rate?
- Counter: How does this affect CPU frequency and power consumption?

---

### 40. What is zero-copy I/O in driver context?
- Counter: What is the difference between `sendfile` and `splice` at driver level?
- Counter: How does `mmap` achieve zero-copy between driver and user space?
- Counter: What is DMA-BUF framework — how does it enable zero-copy across drivers?
- Counter: What is `get_user_pages` — what does it do and what are the risks?
- Counter: What is `vm_insert_page` vs `remap_pfn_range` for zero-copy mmap?

---

## SCENARIO / SYSTEM DESIGN QUESTIONS

### 41. Design a camera driver for an automotive ADAS system
- Counter: How many interrupts per frame at 60fps 4K — can ISR handle it?
- Counter: How do you guarantee frame delivery latency < 5ms?
- Counter: How do you handle V4L2 framework integration?
- Counter: Multiple cameras on same SoC — how do you arbitrate DMA bandwidth?
- Counter: What is CSI-2 (Camera Serial Interface) — driver implications?

---

### 42. Your production device driver causes random silent data corruption
- Counter: No error logs, no crash — where do you start?
- Counter: How do you differentiate hardware bug from driver bug?
- Counter: What is `CONFIG_DEBUG_PAGEALLOC` — how does it help?
- Counter: How do you add integrity checks without impacting performance?
- Counter: Silent corruption only on specific SoC revision — what now?

---

### 43. Design a secure key storage driver using TrustZone
- Counter: What is the driver interface between Normal World and Secure World?
- Counter: How do you prevent key material from ever appearing in Normal World memory?
- Counter: What is OP-TEE — how do you write a TA (Trusted Application)?
- Counter: What is secure memory carveout — how is it enforced in driver?
- Counter: How do you handle secure driver update without compromising stored keys?

---

### 44. You must support your driver on 5 different SoCs with same IP block but different base addresses and IRQ numbers — how do you architect it?
- Counter: What is the role of device tree vs platform data?
- Counter: How do you abstract SoC-specific quirks cleanly?
- Counter: What is `of_device_id` `data` field — how do you use it for per-SoC config?
- Counter: How do you handle IP block version differences in same driver?
- Counter: What is hwspinlock — do you need it for multi-SoC shared resources?

---

### 45. Explain your most complex driver bug you have ever debugged
*(Open-ended — interviewer will probe every technical detail you mention)*
- Counter: Why did that specific locking strategy fail?
- Counter: How did you prove it was the root cause and not a symptom?
- Counter: What would you design differently to prevent the class of bug?
- Counter: How long did it take and what would have reduced that time?
- Counter: Did you upstream the fix — what was the review process?

---

## GRADING REFERENCE (For Self-Assessment Only)

| Level | Expected Depth |
|-------|----------------|
| Basic | API knowledge + why behind each API |
| Intermediate | Driver subsystem internals + design decisions |
| Advanced | Race conditions, memory model, PM, DMA architecture |
| Expert | System-level design, production debugging, upstream contribution |

---

*MAANG device driver interviews test not just API knowledge but system thinking, failure modes, and production experience.*
*If you can answer all counter questions without hesitation — you are ready.*