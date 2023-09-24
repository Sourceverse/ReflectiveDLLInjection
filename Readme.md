关于
========
反射式DLL注入是一种库注入技术，它利用了反射编程的概念，将一个库从内存加载到宿主进程中。因此，库负责通过实现一个最小的可移植可执行文件（PE）文件加载器来加载自身。然后，它可以在最小与宿主系统和进程的交互下，控制如何加载和与宿主进行交互。

该注入技术适用于从Windows NT4到Windows 8，以及x86、x64和适用的ARM平台。

概览
========
远程将一个库注入到进程的过程分为两步。首先，你希望注入的库必须被写入目标进程的地址空间中（以下简称为宿主进程）。其次，需要以一种使库的运行时期望得到满足的方式将库加载到宿主进程中，例如解析其导入项或将其重定位到合适的内存位置。

假设我们在宿主进程中拥有代码执行权限，并且希望注入的库已经被写入到宿主进程内存的任意位置，反射式DLL注入的工作原理如下：

执行权被传递给库的ReflectiveLoader函数，可以通过CreateRemoteThread()或一个小型引导shellcode来实现，这个函数是库的导出表中的一个导出函数。
由于库的映像当前存在于内存的任意位置，ReflectiveLoader首先会计算自己映像在内存中的当前位置，以便能够稍后解析自己的头部。
ReflectiveLoader然后解析宿主进程的kernel32.dll导出表，以计算加载器所需的三个函数的地址，即LoadLibraryA、GetProcAddress和VirtualAlloc。
ReflectiveLoader现在将分配一块连续的内存区域，用于加载自己的映像。由于加载器稍后将正确重定位映像，所以位置并不重要。
库的头部和段被加载到内存的新位置。
ReflectiveLoader然后处理新加载的映像的导入表，加载任何附加的库并解析其各自导入的函数地址。
ReflectiveLoader然后处理新加载的映像的重定位表。
ReflectiveLoader然后调用其新加载的映像的入口点函数DllMain，参数为DLL_PROCESS_ATTACH。库现在已经成功加载到内存中。
最后，ReflectiveLoader将执行权返回给最初调用它的引导shellcode，或者如果是通过CreateRemoteThread调用的，则线程将终止。

构建
========
在Visual Studio C++中打开“rdi.sln”文件，并在Release模式下构建解决方案，以生成inject.exe和reflective_dll.dll。

用法
========
使用inject.exe将reflective_dll.dll注入到宿主进程中，通过进程ID进行注入，例如：

inject.exe 1234

许可证
========
根据3条款的BSD许可证授权，详细信息请参见LICENSE.txt。
