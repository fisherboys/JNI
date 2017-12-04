通过javah命令生成的JNI类型头文件帮助你写本地方法的C或C++实现。你写的函数必须符合生成头文件中的函数原型。你可以在C文件HelloWorld.c中实现HelloWorld.print方法：

```
#include <jni.h>
#include <stdio.h>
#include "HelloWorld.h"

JNIEXPORT void JNICALL
Java_HelloWorld_print (JNIEnv *env, jobject obj)
{
    printf("Hello World!\n");
    return;
}
```

这个本地方法的实现非常简单。它使用printf函数在界面上显示字符串"Hello World!"并返回。正如之前提到的，两个参数JNIEnv指针和对象的引用都忽略了。
这个C程序包含三个头文件：

* jni.h —— 这个头文件包含本地代码需要调用的JNI函数。当写本地方法的时候，你必须在C或C++源文件中包含这个文件。
* stdio.h —— 这个文件包含printf函数。
* HelloWorld.h —— 这个头文件由javah命令生成，里面包含了Java\_HelloWorld\_print函数的C\/C++原型。