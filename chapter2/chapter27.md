## 2.7 运行程序

就此，你有两个就绪的组件运行程序。类文件（HelloWorld.class）调用一个本地方法，本地库（libHelloWorld.so）实现本地方法。  
因为HelloWorld类包含它自己的主函数，你可以在Solaris或者Win32系统中执行如下命令运行程序：

```
java HelloWorld
```

你将看到如下输出：

```
Hello World!
```

如果要运行程序，正确设置本地库的路径非常重要。本地库路径是在加载本地库的时候，Java虚拟机搜索的一个路径列表。如果你没有正确设置本地库路径，那么你将遇到类似下面这种错误：

```
java.lang.UnstatisfiedLinkError: no HelloWorld in library path
    at java.lang.Runtime.loadLibrary(Runtime.java)
    at java.lang.System.loadLibrary(System.java)
    at HelloWorld.main(HelloWorld.java)
```

请确保生成的本地动态库放在其中一个本地库路径下。在Solaris系统中，LD\_LIBRARY\_PATH环境变量的值定义本地库路径。确保这些路径包含libHelloWorld.so文件所在的目录。如果libHelloWorld.so文件在当前目录，你可以通过在命令行中执行如下命令设置LD\_LIBRARY\_PATH环境变量的值：

```
LD_LIBRARY_PATH=.
export LD_LIBRARY_PATH
```

在C shell中，相应的命令为：

```
setenv LD_LIBRARY_PATH .
```

如果你在Windows95或者Windows NT的机器上运行，确保HelloWorld.dll在当前路径下，或者在PATH环境变量所列的路径下。  
在Java 2 SDK1.2稳定版中，你可以在java命令行中使用如下命令指定本地库的路径：

```
java -Djava.library.path=. HelloWorld
```

"-D"选项设置Java平台的系统属性。设置java.library.path属性为“.”表示Java虚拟机会去当前路径搜索本地库。

