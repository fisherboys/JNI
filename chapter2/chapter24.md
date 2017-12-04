接着我们用javah工具生成JNI头文件，它在实现C语言本地方法时会很有用。你可以在HelloWorld类所在的目录执行命令：

```
javah -jni HelloWorld
```

头文件的名字是类名加".h"结尾。上面的命令将产生"HelloWorld.h"文件。我们在这不列出头文件的所有内容。头文件中最重要的部分为Java\_HelloWorld\_print的函数声明，即C语言实现的HelloWorld.print方法：

```
JNIEXPORT void JNICALL
Java_HelloWorld_print (JNIEnv *, jobject);
```

暂时先忽略JNIEXPORT和JNICALL两个宏。你会发现这个本地方法的C语言实现接收两个参数，虽然相应的本地方法声明中没有参数。第一个参数一个JNIEnv接口指针。第二个参数是HelloWorld对象本身的引用\(类似C++中的"this"指针\)。我们将在本书的后面部分讨论如何使用JNIEnv接口指针和这个jobject参数，但在这个例子中，忽略他们即可。