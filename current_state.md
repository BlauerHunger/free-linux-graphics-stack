# The current state of the Linux graphics stack
The Linux graphics stack consists of different parts, some of them are already free, but some are proprietary and cause problems. 

## Hardware drivers (kernel modules)

### Intel
Intel only has an open-source driver for Linux, which uses Mesa and DRI and thus is well integrated in almost every software which has to deal with this part of a system. 

### AMD
AMD graphics card owners used to have two options which driver they want to use, *radeon*, the free driver, or *fglrx*, the proprietary driver, also known as catalyst. Users of the free driver had issues with Mesa, the OpenGL library used by radeon, which performed a lot worse than the OpenGL implementation of fglrx. 

Early 2015 AMD dropped support for fglrx and announced their new *amdgpu*-driver. amdgpu is a free kernel module, which can either use Mesa or a proprietary variant, known as AMDGPU-PRO, for OpenGL and Vulkan. amdgpu supports graphics cards starting with GCN1.0, users of older cards need to use radeon, since fglrx isn't compatible to new X.org-Xservers and new kernels. amdgpu is like the Intel driver well integrated in almost every software used in this area. 
The result of this work is that everyone with an AMD graphics card and Linux uses a free kernel driver now. 

### Nvidia
Nvidia graphics card owners also have a choice which driver to use, there is *nouveau*, the free driver, and *nvidia*, the proprietary driver. nouveau uses Mesa for 3D, while nvidia ships its own library for that. 

Nvidia not only doesn't put manpower into nouveau, but also actively prevents nouveau from getting the same performance out of a graphics card like the proprietary driver by requiring a nvidia-signed firmware to be loaded by the driver for reclocking, e.g. raising the clock speeds of the gpu core and memory to archieve the performance you would expect from a graphics card, starting with the Maxwell microarchitecture (GTX 9XX series). This leads to nouveau being only usable for outputting video to the screen, gaming or rendering is impossible. Also it takes a long time for nouveau to use 3D acceleration (even without reclocking) because they get their information about Nvidia cards only from reverse engineering. 
There are efforts in Nvidia's Linux drivers team to switch to a strategy like AMD's, but this is blocked completely by their management. 

### Others
There are kernel modules for other graphics devices available, mostly virtual ones like the QXL video card from QEMU. They are used very similar to amdgpu and the Intel driver and in all common cases free software and shipped with the Linux kernel itself. 

## 3D libraries
This is the part between the applications, e.g. video editors or games and the kernel modules. They should provide OpenGL, Vulkan, OpenCL and maybe some other interfaces. 

### Mesa
Mesa is the 3D library used by all free drivers. It consists of *Gallium3D*, a libary, which is used by applications, when they want to draw 3D graphics on the screen. It talks to the kernel modules via the *Direct Rendering Infrastructure* (DRI). This is done by device-specific libraries, which convert the OpenGL or Vulkan commands into commands the graphics card can handle. Examples for these device-specific libraries are *radeonsi* for new AMD graphics cards, *r600* for older AMD cards, nvc0 for modern Nvidia GPUs or i965 for Intel's graphics chips. There are also software renderers available, namely *softpipe* and *llvmpipe*. 
Mesa provides OpenGL, Vulkan, OpenCL and Direct3D 9 (the last thanks to the Gallium Nine project). 

Originally, Mesa was backed almost only by Intel, but when starting work on amdgpu, AMD's developers also contributed more and more to Mesa. Also Valve, the company behind the game distribution platform Steam, games like Half Life, Counter Strike, Left4Dead, Dota and Team Fortress and the Source Engine, hired Mesa developers. 

### AMDGPU-PRO
AMDGPU-PRO is the AMD's proprietary graphics stack and it sits on top of the amdgpu kernel module. It provides OpenGL, Vulkan and OpenCL (the OpenCL component can also be used together with Mesa). 
AMDGPU-PRO currently provides only a little better performance than Mesa because AMD works hard on making Mesa better. Valve has also dropped AMDGPU-PRO from SteamOS, probably because the free stack with Mesa is easier to maintain. 

### Nvidia
Nvidia ships their own OpenGL and Vulkan implementation. Their OpenGL is currently the most efficient available on Linux, and it only works with the proprietary kernel driver. 

## Special use cases

### Hybrid graphics
This technology is primarily marked by Nvidia under the term *Nvidia Optimus*, but also known as *PRIME GPU offloading*. It is implemented in notebooks with dedicated GPUs and causes big problems there, which also led Linux Torvalds to say "Nvidia, f*ck you". 

A PRIME setup consists of an Intel integrated graphics chip (iGPU), which drives the notebook screen and is used to draw the desktop and office applications. And there is a dedicated GPU (dGPU) which is used to render complex things like games while the output still goes through the iGPU. The display outputs like HDMI or DisplayPort are hardwired sometimes to the iGPU, but in most cases to the dGPU. There are two solutions for this: 


**PRIME** is a feature in Mesa which enables the use of hybrid graphics with free drivers. Applications run with the environment variable DRI_PRIME=1 are offloaded to the dGPU, and RandR sees the displays wired to the dGPU. Originally, there was some setup required, but starting with DRI3 this works out of the box. Since it's a part of Mesa, it only works with the free drivers, which leads to crippled performance on Nvidia cards. 

**Bumblebee** is a software suite consisting of a daemon, *bumblebeed*, some utilities, *optirun* and *primusrun*, and a kernel module, *bbswitch*. When you start an application using primusrun or optirun, it starts up the dGPU using bbswitch, loads the driver (usually the proprietary Nvidia driver, since you don't need this otherwise), fires up an Xserver forwarding its output to the one running on the iGPU and running the requested application on it. This leads to additional CPU overhead and usually has issues with outputs hardwired to the dGPU, but is the only way of using proprietary drivers. Sadly, the project is almost dead since 2013.
