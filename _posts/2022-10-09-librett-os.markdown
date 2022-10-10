# Libertt OS

Oct 9, 2022

### Gist of the Blog

**Monolithic OS** has several shortcomings be it **performance**, **security** concerns of modules in **kernel address space**, or failure to **recover** from a fault/crash. **Libertt OS** tries to address these issues by delegating some traditional **kernel responsibilities** as **services** in the user space, allowing applications to dynamically switch to between physical and virtualized hardware interface on runtime, recoverability, runtime upgrades and much more…

### Here are a few things to discuss before setout

#### What's Kernel?
Kernel as the name implies, is the core part of Operating System that has virtually complete control over the system. Over the years, different kernel architechture evolved aimed at assigning responsibilities of a kernel.

#### What is a Monolithic Kernel then?
![Monolithic Kernel](https://wiki.osdev.org/images/a/aa/Monolithic.png)

Kernel tries to provide every conceivable service(a bit of exaggeration, of course) as an ingrained generic driver/module under one bundle. This can quickly lead to bloated code, which isn't tailored to meet application quirks. With a large surface area, attackers can exploit poorly designed modules. Furthermore, overarching responsibilities and promises inhibit the ability to redesign and integrate new ideas.
![Problems](https://raw.githubusercontent.com/kanwaekan/kans-blog/main/6wcl94.jpg)



#### Does Microkernel save the day?
![Microkernel](https://wiki.osdev.org/images/2/28/Microkernel.png)

Less on steroids, take a more conservative approach in undertaking kernel responsibilities. Microkernel provides services such as **device drivers** used by applications in the userland as **servers**. This leads to the isolation of services, and increased kernel security. Failures of the Driver wouldn't (and should not) affect the kernel, and therefore, it localizes failures to the application using it. Upgrading a Service could be just as easy as shutting down and running the new one in its stead.

So why is it not ubiquitous, then? Could it be Easily Summarized in a meme?
![Toni! Toni!](https://raw.githubusercontent.com/kanwaekan/kans-blog/main/6wal21.jpg)

Again, a bit of exaggeration, but basically it tries to convey that, interprocess communication has a penalty; if the apps try to communicate with multiple services (which, in turn, might use IPC themselves) very frequently(like I/O processes), the amount of useful work carried out by it is less.


#### Kernel Bypass and LibOS
![DPDK](https://raw.githubusercontent.com/kanwaekan/kans-blog/main/6we123.png)

Eliminates layers of abstraction and indirection by directly allowing the application to interface with hardware. Interrupts are now directly passed to the userspace without any interrupt processing, and CPU mode switch isn't necessary since application owns resource exclusively.


Libraries aren't conformed to any standards thus can be better designed to cater to application specific needs,  but this requires a massive engineering overall whenever API/ABI breaks. Therefore, it vital for applications to adhere to a standardized interface such as ones defined in POSIX in order to maintain compatibility and have the flexibility to augment features that can't be incorporated.

Requires kernel to undertake the responsibility of a resource arbiter.

#### Unikernel
![Rumprun](https://github.com/kanwaekan/kans-blog/blob/main/rumparch.png)

Unikernel does not have any bloat of the traditional operating system and provides an minimalistic environment and a runtime with a singlular address space for applications to run. Single Address avoid mode switches between user and kernel modue. They're bundled with application along the libOS which, drastically reduces dependencies required unlike a traditional operating system which is intended to be multipurpose; has support for almost all. 

Thus it provides us a viable candidate to develop suite of application isolated from each other, sharing different implementation of underlying libraries all the while lacking the bloat of the traditional VM instance.

### Overarching Idea

LibOS uses microkernel architecture at its core by separating core drivers as user-space modules. Application get services from using dedicated servers or directly use the underlying hardware.

### Implementation Details

#### SR-IOV(Single Root Input/Output Virtualization):
![SR-IOV](https://honser.github.io/images/sriov_overview2.png)

An extension to PCI-E specification allows PCI-E resources to advertise themselves as a set of Multiple Virtual Interfaces providing Dedicated Isolated. SR-IOV provides two PCI functions:

- Physical Function(PF) provides functionalities of a conventional PCI-E while providing a mechanism to manage SR-IOV functionality.
- Virtual Function(VF) can function as a normal PCI-E device that can restrict one or more functions; unlike software virtualization, hardware-assisted virtualization provides dedicated buffers, _application_ mapped interrupts and DMA streams.

Virtual Interfaces are limited in number, and must be used conservatively. The Kernel manages devices through Physical Function and allocates Virtual Device to Applications.

#### Rump Kernel
![Rumpkernel](https://raw.githubusercontent.com/kanwaekan/kans-blog/main/rumparch.png)

Rump Kernel is the Unikernel discussed, for POSIX compatibility, application receives call from program through libc which is handled by rump kernel. Rump kernel is the conceptual fruit of anykernel concept which aims for creating drivers that is OS agnostic.
The Kernel interface with the hardware through rumprun.

#### IOMMU

IOMMU is provides hardware security in addition to the numerable features it provides. PCI-E device are mapped to VMs which disallows out-of-bound memory access and interrupts. We route SR-IOV's virtual function in tandem with IOMMU.

### Design

![Design](https://raw.githubusercontent.com/kanwaekan/kans-blog/main/6wd124.png)
Libertt OS
POSIX Compatibility
- Existing NetBSD drivers were ported are ran as rumpinstances, they are the *servers* in the microkernel.
- Application are either provided with Virtual Interface or access to a PCI-E hardware through virtual function. Dynamic Switch involves support from the libOS and the microkernel/hypervisor without causing any noticiable interruption in application and service.
- Applications are run under a rumprun instance Isolated from each other under shared hardware, usage of libc allows for a greater degree of compatability while under the hood, receiving benefits of a libOS.
- Applications under the discretion of the kernel can directly interface with hardware using VFs or indirectly using a service under a virtual interface. This is exchange of information happens through IPC.
- A Hypervisor assumes the role of a manager which apart from setting up and monitoring individual rumpinstances, provides access to Virtual Functions and aids in dynamic mode switch by maintain global data of IPs, interfaces etc.
- Rumprun instances schedule its process under a Virtual CPU, which in turn is scheduled by the hypervisor.


