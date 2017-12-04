当你在HelloWorld.java文件中创建HelloWorld类的时候，你添加了一行代码，将本地库加载进程序中：
  ```
  System.loadLibrary("HelloWorld");
  ```

  既然所有必要的C代码都已完成，你需要编译HelloWorld.c并创建本地库。
  不同的操作系统支持不同的方式创建本地库。在Solaris系统中，下面的命令将生成一个名为libHelloWorld.so的动态库：
  ```
  cc -G -I/java/include -I/java/include/solaris
    HelloWorld.c -o libHelloWorld.so
  ```

  -G编译选项告诉C编译生成一个动态库而不是Solaris的可执行文件。由于篇幅原因，将命令分成两行。你需要在同一行中输入上述命令，或者将命令写入一个脚本文件中。在Win32系统中，下列命令用微软的Visual C++编译器生成一个动态链接库HelloWorld.dll：
  ```
  cl -Ic:\java\include -Ic:\java\include\win32
    -MD -LD HelloWorld.c -FeHelloWorld.dll
  ```

  -MD编译选项保证HelloWorld.dll链接到win32的多线程C库。-LD编译选项告诉C编译器生成一个动态库而非可执行文件。当然，在Solaris和Win32中，include路径都需要与你机器上的路径一致。

---

以下为译者添加，非本书中的内容，仅供参考。

在linux系统中，可以通过gcc命令生成动态库：

```
gcc -I/java/include/ -shared -o libHelloWorld.so
```

shared:表示动态库
-I：头文件所在的实际路径，需要与你机器上的路径一致

---
