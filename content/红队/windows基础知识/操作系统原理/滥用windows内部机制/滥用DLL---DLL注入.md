![[image-12.png]]

分解一个基本的 DLL 注入器来识别每个步骤，并在下面更深入地解释。

1--DLL注入的第一步，我们必须找到目标线程。可以使用三个 Windows API 调用从进程中定位线程： `CreateToolhelp32Snapshot()` 、 `Process32First()` 和 `Process32Next()` 。

2--第二步，枚举完 PID 后，我们需要打开进程。这可以通过各种 Windows API 调用来完成： `GetModuleHandle` 、 `GetProcAddress` 或 `OpenProcess` 。

3--第三步，必须为所提供的恶意 DLL 分配内存以驻留。与大多数注入器一样，这可以使用 `VirtualAllocEx` 来完成。

4--第四步，我们需要将恶意DLL写入分配的内存位置。我们可以使用 `WriteProcessMemory` 写入分配的区域。

5--第五步，我们的恶意 DLL 被写入内存，我们所需要做的就是加载并执行它。要加载 DLL，我们需要使用 `LoadLibrary` ；从 `kernel32` 导入。加载后， `CreateRemoteThread` 可用于使用 `LoadLibrary` 作为启动函数来执行内存。


综上，为dll注入的详细过程，dll注入作为一种常用和重要的攻击方法，一定要多去研究和积累