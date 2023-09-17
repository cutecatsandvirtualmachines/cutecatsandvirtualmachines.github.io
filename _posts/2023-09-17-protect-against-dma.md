## Protect your software from DMA access

As you're probably aware, cheating is a [recurring problem](https://www.essentiallysports.com/esports-news-cheating-and-racism-allegations-rock-two-hundred-fifty-thousand-dollar-fortnite-tournament/).

One of the most widely used methods for cheating, especially in private leagues and tournaments, is by using DMA capable devices with custom firmwares
that are able to read and write the game's memory.

### The current "meta"
All major anti-cheats currently use the same approach to contrast DMA cheating:

signature scans on the PCIE config space

As we will see, there's definetely a much better solution to this problem, without any of the related [downsides](https://www.reddit.com/r/ValorantTechSupport/comments/wqcc93/valorant_turning_off_internet_every_time_it_opens/)

### The new "meta"
I have been researching into DMA cheating for a while now, and whilst my knowledge is not by any means complete, I do have some interesting insights.

Particularly DMA devices are a security concern since the first inception of the VM technology (even before EPT/NPT was a thing!), as they
are capable of bypassing virtualizations systems on the CPU, given they are "directly" connected to the CPU.

To compensate this issue, motherboard manufacturers now include a new component called (usually) IOMMU -> IO memory managment unit.

This component sits in between the memory and the PCIE device, and is capable of various interesting things:

1. Interrupt virtualizations
2. Memory access remap
3. Other virtualization stuff

What we care about in particular, is point 2.

### Memory access remapping
Pretty much like SLAT, DMA remapping is used by virtualizers (usually) to make sure devices are not able to access memory from the host, or other virtual machines.

It's very simple as it is just a set of page tables that translate device physical addresses to host physical addresses, and can accesses can be redirected or blocked.

In my example [here](https://github.com/cutecatsandvirtualmachines/DmaProtect) the approach used is to create a set of page tables that describe basically a 1:1 mapping with host's physical memory.

Then at runtime take a bunch of these tables and modify their flags to point to a different physical address.

#### How it works
Let's say a DMA capable device tries to access address 0x1000, and it is protected. In my example driver said address is set to redirect to 0x4000 (random address chosen as example),
so what will happen is that the device reads 0x1000, but the read contents are of 0x4000.

This translation is transparent to the device like page tables are transparent to user mode programs.

#### Implementation problems
One common bypass for this relies on very simply modifying the page tables to not protect the address anymore. The fix is **not** implemented in my sample, but is trivial to do so.

Just make sure to have your page tables inside the driver's .data section, and then use a 2MB table to protect itself and other tables. It's easier said than done :)

### Who can benefit from this?
Everyone but cheaters, especially DMA cheaters. This technology can be used by anti-cheats to protect their games. The only requirement is to lock the protect pages in RAM
by using VirtualLock (or NtVirtualLock in kernel, not exported).

### Why are you posting this?
I hate DMA providers, have a good rest of your lives without your pasteware :)

### Contact me
If you are a researcher or a representative for an anti-cheat company, and are interested in working with, or hiring me, feel free to contact me on my brand new email:
cutecatsandvirtualmachines@gmail.com
