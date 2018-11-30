# KVM Code Navigate

*arch/x86/kvm/svm.c*

`svm_init(void)`  简单调用kvm_init函数对kvm模块进行初始化，传入`kvm_x86_ops`结构体变量`svm_x86_ops`。`kvm_x86_ops`定义了kvm实现虚拟化时需要进行的所有操作，如`cpu_has_kvm_support, vcpu_create, vm_alloc, get_msr, run`等。

- - -
*virt/kvm/kvm_main.c*
`int kvm_init(void *opaque, unsigned vcpu_size, unsigned vcpu_align, struct module *module)`完成整个KVM初始化工作。
```
 r = kvm_arch_init(opaque);
 r = kvm_irqfd_init();
```
`kvm_arch_init`函数首先判断kvm是否已经被初始化过，如果没被初始化则再判断当前CPU是否支持kvm虚拟化，即CPU是否支持硬件虚拟化如*Intel VT-x/AMD-V*，然后在判断当前BIOS是否使能硬件虚拟化。CPU硬件是否支持硬件虚拟化技术通过`cpuid`指令来判断，而BIOS是否使能硬件虚拟化则通过读MSR寄存器相应标志位来判断。所有判断完成后，便开始真正的架构相关初始化操作。首先调用`alloc_percpu(struct kvm_shared_msrs)`来分配guest host共享的msr percpu数据，`struct kvm_shared_msrs`共定义了`KVM_NR_SHARED_MSRS （16）`个共享msr寄存器。

然后调用`kvm_mmu_module_init(void)`函数对kvm mmu相关数据结构进行初始化操作，该函数首先创建了`struct pte_list_desc`和`struct kvm_mmu_page`两个kmem_cache并赋值给对应的全局变量`pte_list_desc_cache`和`mmu_page_header_cache`。之后将`kvm_total_used_mmu_pages`的percpu初始化为0，初始化过程包括对percpu数据内存的分配。最后注册mmu_shrinker，shrinker可参考如下链接[shrinker, a mechanism by which the memory management subsystem can request that cached items be discarded and their memory freed for other uses](https://lwn.net/Articles/550463/)


接下来调用`kvm_set_mmio_spte_mask`函数，对mmio shadow pte相关mask的全局变量进行初始化，包括`shadow_mmio_value`和`shadow_mmio_mask`两个全局变量。接着为全局变量`kvm_x86_ops`进行赋值后，调用`kvm_mmu_set_mask_ptes`函数对shadow pte相关mask进行初始化，包括`shadow_user_mask, shadow_accessed_mask, shadow_dirty_mask`等。

`kvm_timer_init`函数主要对TSC时钟相关进行初始化。该函数首先设置`max_tsc_khz`，然后判断当前处理器是否支持常量TSC，如果处理器不支持常量TSC，则操作系统对CPU频率进行调节的时候会影响TSC的频率，所以需要KVM进行特殊处理，保证客户机时钟的准确性。KVM对可变TSC处理的方式是向系统注册相关notifier，当系统调整CPU频率时会调用注册的notifier，**具体处理方式另述**。最后调用`cpuhp_setup_state`来注册CPU热插拔相关回调函数`kvmclock_cpu_online`和`kvmclock_cpu_down_prep`。




## Wht is Eventfd and Irqfd






