# Libertt OS

Oct 9, 2022

### Gist of the Blog

**Monolithic OS** has several shortcomings be it **performance**, **security** concerns of modules in **kernel address space**, or failure to **recover** from a fault/crash. **Libertt OS** tries to address these issues by delegating some traditional **kernel responsibilities** as **services** in the user space, allowing applications to dynamically switch to **dedicated libraries**(libOS) that directly interact with physical hardware on runtime, recoverability, runtime upgrades and much more…

### Here are a few things to discuss before setout

#### What is a Monolithic Kernel?
![Monolithic Kernel](https://wiki.osdev.org/images/a/aa/Monolithic.png)

OS tries to provide every conceivable service(a bit of exaggeration, of course) as an ingrained generic driver/module under one bundle. This can quickly lead to bloated code, which isn't tailored to meet application quirks. With a large surface area, attackers can exploit poorly designed modules. Furthermore, overarching responsibilities and promises inhibit the ability to redesign and integrate new ideas.
![Problems](https://raw.githubusercontent.com/kanwaekan/kans-blog/main/6wcl94.jpg)



#### Does Microkernel save the day?
![Microkernel](https://wiki.osdev.org/images/2/28/Microkernel.png)

Less on steroids, take a more conservative approach in undertaking kernel responsibilities. Microkernel provides **device drivers** used by applications in the Userland. This leads to the isolation of services, and increased kernel security. Failures of the Driver wouldn't (and should not) affect the kernel, and therefore, it localizes failures to the application using it. Upgrading a Service could be just as easy as shutting down and running the new one in its stead.

So why is it not ubiquitous, then? Could it be Easily Summarized in a meme?
![Toni! Toni!](https://raw.githubusercontent.com/kanwaekan/kans-blog/main/6wal21.jpg)

Again, a bit of exaggeration, but basically it tries to convey that, interprocess communication has a penalty; if the apps try to communicate with multiple services (which, in turn, might use IPC themselves) very frequently(like I/O processes), the amount of useful work carried out by it is less.


#### Service Bypass

Eliminate layers of abstraction on indirection by directly allowing the application to interface with hardware. However, handling multiple applications coordinating multiple applications to use the same hardware requires support from the hardware. See SR-IOV. Should we have to handcraft drivers for our application, then?

#### LibOS to the Rescue

Services provided by libraries that directly interface with the hardware, applications can own or share libraries amongst themselves, and libraries are better designed to cater to application specific needs. This minimizes the kernel's direct intervention and the amount of indirection but can lead to a less standardized interface that requires massive engineering effort. Requires kernel to undertake the responsibility of a resource arbiter.

#### Unikernel

Unikernel does not have any bloat of the traditional operating system and provides an environment for applications to run. Due to the nature of its conservative design, smaller role. It promises better performance and security.

### Overarching Idea

LibOS uses microkernel architecture by separating core drivers as user-space modules.

### Implementation Details

#### SR-IOV(Single Root Input/Output Virtualization):

An extension to PCI-E specification allows PCI-E resources to advertise themselves as a set of Multiple Virtual Interfaces providing Dedicated Isolated. SR-IOV provides two PCI functions:

- Physical Function(PF) provides functionalities of a conventional PCI-E while providing a mechanism to manage SR-IOV functionality.
- Virtual Function(VF) can function as a normal PCI-E device that can restrict one or more functions; unlike software virtualization, hardware-assisted virtualization provides dedicated buffers, _application_ mapped interrupts and DMA streams.

Virtual Interfaces are limited in number, and must be used conservatively.

#### Anykernel & Rump Kernel

Drivers are designed to run standalone and not tied to any OS Architecture. Rump Kernel provides a minimalistic environment for services to be compiled and run on top of a lightweight kernel. Is Archit N:1 non-preemptive scheduler NetBSD Drivers

#### IOMMU

You would have probably heard about Memory Management Unit when discussing the Process's Virtual Address Translation to Physical Address. Well, the same could be said for IOMMU which

### Design

Libertt OS

- Applications are run under a rumprun instance Isolated from each other under shared hardware.
- Applications under the discretion of the _kernel_ can directly interface with hardware using VFs or indirectly using a proxy service under a virtual interface.
- Scheduling: Rumprun instance can schedule its process under a Virtual CPU, which in turn is scheduled by the hypervisor.
- Maintains POSIX/BSD API calls to avoid extensive reengineering efforts.

### References:

- https://www.lynx.com/embedded-systems-learning-center/what-is-sr-iov-and-why-is-it-important-for-embedded-devices
- https://learn.microsoft.com/en-us/windows-hardware/drivers/network/overview-of-single-root-i-o-virtualization–sr-iov-

##
