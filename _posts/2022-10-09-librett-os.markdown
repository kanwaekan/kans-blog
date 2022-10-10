---
layout: post
title:  "Libertt OS"
---



### Gist of the Article
Monolithic OS has several shortcomings be it performance, security of modules running in kernel space, failure to recover from a faulty execution. Libertt OS provides addresses these shortcomings by delegating  kernel responsibilty as services in the user space, allowing applications to dynamically switch to dedicated libraries that directly interact with phyiscal hardware on runtime, recoverability, runtime upgrades and much more...

### A Few things to discuss before we set out 
#### What's a Monolithic Kernel?
OS tries to provides every concevable service(a bit of an exaggeration ofcourse) as an ingrained generic driver/module under one bundle. This can quickly lead to bloated code, which mightn't be tailored to meet certain Application demands. With a large surface area, attackers are be able to exploit poorly designed modules. And Overarching responsibilties and promises inhibit the ability to redesign and integrate new ideas.

#### Is Micro Kernel our saviour?
Less on steroids, takes more conservative approach in undertaking kernel responsibilties. Provides Device Drivers used by Application in the Userland. Therefore leading to isolation of services and kernel increasing security. Failures of Driver would't(and shouldn't) affect the kernel and therefore localizing failures to application using it. Upgrading a Service could be just as easy as shutting down and running the new one in its stead.

So why is it not ubiquitous then? Could be Easily Summarized in a meme?

Again a bit of exaggeration, but basically interprocess communication has a penalty to it, if your apps tries to communicate with multiple services (which inturn might use ipc themselves) very frequently(like I/O processes), the amount of useful work carried out it less. 

#### Kernel Bypass
Eliminate layers of abstraction on indirection by bypassing kernel calls and allowing application to directly interface with hardware. But, handling coordinating multiple application to use the same hardware requires support from hardware. See SR-IOV. Should I have to handcraft drivers for my Application then?

#### LibOS to the Rescue
Services are provided by libraries that directly interface with the hardware, application can own or share libraries amongst themselves, libraries are designed to cater to specific application needs. This minimizes kernel's direct intervention, amount of indirection, but can lead to less standardized interface that requires massive engineering effort. 
Requires Kernel to undertake the responsibilty of a resource arbiter.


And Finally, 

#### Unikernel
Doesn't have any bloat of the traditional operating system, provides an minimalisitc environment for application to run.
Applications are packaged with LibOS. Since, it applications interacts with library calls directly which in turn owns the access to hardware resources , it doesn't need context switch.
Due the nature of it's conservative design, smaller role. it promises better performance and security.


### Overarching Idea
LibrettOS uses microkernel architecture, by seperating services used by application from the kernel and running them as standlone servers\*. Applications under the discretion of kernel/hypervisor can interact with the server with the help of IPC or the hardware.

### Implementation Details
#### SR-IOV(Single Root Input/Output Virtualization):
An extension to PCI-E specification that allows PCI-E resource to advertise itself as set of Multiple Virtual Interfaces providing Dedicated Isolated.
SR-IOV has provides two PCI functions:
- Physical Function(PF) provides functionalaties of a conventional PCI-E whilst providing mechanism to mange SR-IOV functionality.
- Virtual Function(VF) can function as a normal PCI-E device can be restrict one or more functions, unlike software virtualization, hardware assisted virtualization provides dedicated buffers, *application* mapped interrupts, and DMA streams.

Virtual Interfaces are limited in number. Thus, must be used conservatively. It's provided to Application that's deemed worthy, don't contain malicious code and would benefit from its usage else, it'ss shared among application through a Virtual Interface by using a dedicated service which can provide additional security checks. Best of both worlds eh?

#### IOMMU
Maps Interrupts and I/O address used by devices to VMs. Aids in hardware-assisted memory protection both ways by preventing devices injecting unassigned interrupts or disallowing both devices and VMs using out-of-bound DMA operations/addresses.
Provides safe PCI-E passthrough to access Virtual Interface.

#### Anykernel & Rump Kernel
Drivers are designed to ran standlone and not tied any OS Architecture.
Rump Kernel provide a minimalistic environment for services to be compiled and ran in top of a lightweight kernel. Is Archit
N:1 non-preemptive scheduler
NetBSD Drivers

#### Design
Libertt OS 
- Application are ran under a rumprun instance using hardware assited virtualization Isolated from each other under a shared hardware. 
- Applications under the discretion of the *kernel* can directly interface with hardware using VFs or by indirectly using a proxy service under a virtual interface.
- Dynamic Mode Switch: Allows Application to decouple without interruption from using a dedicated server to directly access the hardware. This Transfer typically doesn't require any modifiction to Application but a few features that enables to adapt itself when the switch occurs.
	Based on I/O load, applications can be allowed to switch from servers to directly to interface on hardware
- Runtime Upgrades: Depends on Application and the Service in general but, the ability to swap out an service with an upgraded one without compiling paves way to upgrades that doesn't bring the system to a halt.
- Scheduling: Rumprun instance can schedule its own process under a Virtual CPU which in turn is scheduled by the hypervisor, therefore and M:N shceduling.
- Maintains POSIX/BSD API calls(except fork() and pipe() due implmentation complexitu) to avoid extensive reengineering effort. Allows for multi-threading instead.

### Benchmark

For Detailed report see, the reference.
- 




### References:
- https://www.lynx.com/embedded-systems-learning-center/what-is-sr-iov-and-why-is-it-important-for-embedded-devices
- https://learn.microsoft.com/en-us/windows-hardware/drivers/network/overview-of-single-root-i-o-virtualization--sr-iov-
