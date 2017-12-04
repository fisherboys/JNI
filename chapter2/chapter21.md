## 2.1 概览

图2.1说明使用JDK或者Java 2 SDK编写一个简单的Java应用程序来调用C函数打印"Hello World!"。整个过程由以下步骤组成：

1. 创建一个声明本地方法的类HelloWorld.java。
2. 使用javac命令编译HelloWorld.java源文件得到HelloWorld.class文件。
3. 使用javah -jni命令生成C的头文件HelloWorld.h，头文件中包含本地方法的函数声明。
4. 编写本地方法的C语言实现\(HelloWorld.c\)。
5. 将HelloWorld.c源文件编译进本地库，得到HelloWorld.dll或者libHelloWorld.so。所用的C编译器和链接器在宿主环境中。
6. 在java运行环境中运行HelloWorld程序。类文件\(HelloWorld.class\)和本地库\(HelloWorld.dll或者libHelloWorld.so\)都加载进运行时中。

这章的其余部分将详细说明这些步骤。

![](/assets/Figure_2_1.jpg)

图2.1

