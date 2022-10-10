---
layout: post
title:  "Libertt OS"
---



### Aim
Monolithic OS has several shortcomings be it performance, security of modules running in kernel space, failure to recover from a faulty execution. Libertt OS provides tries to provide best of both worlds by delegating  kernel responsibilty as services in the user space, allowing applications to dynamically switch to dedicated libraries that directly interact with phyiscal hardware on runtime, recoverability and runtime upgrades...

### A Few things to discuss before set out 
#### What's a Monolithic Kernel
OS tries to provides every concevable service(a bit of an exaggeration ofcourse) as an ingrained generic drivers/module under one bundle. This can quickly lead to bloated code, which mightn't be tailored to meet certain Application demands. With a large surface area, attackers are be able to exploit poorly designed modules. And Overarching responsibilties and promises inhibit the ability to redesign and integrate new ideas.

#### What's Micro Kernel?
Less on steroids, takes more Conservative Approach in undertaking kernel responsibilties. Provides Device Drivers used by Application in the Userland. Therefore leading to isolation of services and kernel increasing security. Failures of Driver would't(and shouldn't) after the kernel and therefore localizing failures to application using it. Upgrading a Service could be just as easy as shutting down and running the new one in its stead.

So why is it not ubiquitous then? Could be Easily Summarized in a meme?

Again a bit of exaggeration, but basically interprocess communication has a penalty to it, if your apps tries to communicate with multiple services (which inturn might use ipc themselves) very frequently(like I/O processes), the amount of useful work carried out it less. 

### Way to mitigate
#### Service Bypass
Eliminate layers of abstraction on indirection by directly allowing application to interface with hardware. But, handling multiple application coordinating multiple application to use the same hardware requires support from hardware. See SR-IOV. Should I have to handcraft drivers for my Application then?

#### LibOS to the Rescue
Services are provided by libraries that directly interface with the hardware, application can own or share libraries amongst themselves, libraries are designed to cater to application needs. This minimizes kernel's direct intervention, amount of indirection, but can lead to less standardized interface that requires massive engineering effort. 
Requires Kernel to undertake the responsibilty of a resource arbiter.


#### Unikernel
Doesn't have any bloat of the traditional operating system, provides an environment for application to run.
Due the nature of it's conservative design, smaller role. it promises better performance and security.

### Overarching Idea
LibOS uses microkernel architecture, by seperating core drivers as user space modules

### Implementation Details
#### SR-IOV(Single Root Input/Output Virtualization):
An extension to PCI-E specification that allows PCI-E resource to advertise itself as set of Multiple Virtual Interfaces providing Dedicated Isolated.
SR-IOV has provides two PCI functions:
- Physical Function(PF) provides functionalaties of a conventional PCI-E whilst providing mechanism to mange SR-IOV functionality.
- Virtual Function(VF) can function as a normal PCI-E device can be restrict one or more functions, unlike software virtualization, hardware assisted virtualization provides dedicated buffers, *application* mapped interrupts, and DMA streams.

Virtual Interfaces are limited in number. Thus, must be used conservatively.


#### Anykernel & Rump Kernel
Drivers are designed to ran standlone and not tied any OS Architecture.
Rump Kernel provide a minimalistic environment for services to be compiled and ran in top of a lightweight kernel. Is Archit
N:1 non-preemptive scheduler
NetBSD Drivers
#### IOMMU
You would've probably heard about Memory Management Unit when discussing Process's Virtual Address Translation to Physical Address. Well, the same could be said for IOMMU which 

### Design
Libertt OS 
- Application are ran under a rumprun instance Isolated from each other under a shared hardware. 
- Applications under the discretion of the *kernel* can directly interface with headwa using VFs or by indirectly using a proxy service under a virtual interface.
- Scheduling: Rumprun instance can scheduler its own process under a Virtual CPU which in turn is scheduled by the hypervisor.
- Maintains POSIX/BSD Api calls to avoid extensive reengineering effort.
- 

#### Network Server

#### Dynamic Switch


### References:
- https://www.lynx.com/embedded-systems-learning-center/what-is-sr-iov-and-why-is-it-important-for-embedded-devices
- https://learn.microsoft.com/en-us/windows-hardware/drivers/network/overview-of-single-root-i-o-virtualization--sr-iov-
