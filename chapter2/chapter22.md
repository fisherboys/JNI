从写下面的Java程序开始。这个程序定义了一个名字为“HelloWorld”的类，里面包含一个print方法。

```
class HelloWorld {
    private native void print();
    public static void main(String[] args) {
        new HelloWorld().print();
    }
    static System.loadLibrary("HelloWorld");
}
```

HelloWorld类在开始的时候声明了print\(\)的本地方法。接着在主函数中创建一个HelloWorld对象调用print\(\)本地方法。类的最后部分是一个静态初始化，加载包含实现print\(\)本地方法的本地库。

本地方法的声明与Java语言中的方法声明有两个不同点。本地方法的声明必须包含native关键字。native关键字表明这个方法使用其他语言实现。本地方法的声明也是以分号结束，语句结束符号，因为在类中没有本地方法的实现。我们将在另一个C文件中实现print方法。

在本地方法print被调用之前，实现print的本地库需要先被加载。为此，我们在HelloWorld类初始化的时候静态加载本地库。Java虚拟机在调用任何HelloWorld类中的方法之前自动加载静态块，因此保证在本地方法调用之前本地库已经被加载

我们定义一个主函数运行HelloWorld类。主函数以与调用普通方法相同的方式调用本地方法print。

System.loadLibrary包含一个库名，定位具有相同名字的库名，并将该库加载入应用程序中。我们将在本书的后续章节讨论加载过程。现在仅需记住，为了使System.loadLibrary\("HelloWorld"\)成功加载，我们需要创建一个名为HelloWorld\(在win32平台上\)或libHelloWorld.so\(在Solaris平台上\)的本地库。