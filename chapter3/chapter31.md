## 3.1 一个简单的本地方法

我们先从一个简单的例子开始，这个例子与上一章中的HelloWorld程序没有太大的区别。这个样例程序\(Prompt.java\)包含一个本地方法打印一段字符串，等待用户的输入并且返回用户输入的行数。这个程序的源码如下所示：

```
class Prompt {
    //native method that prints a prompt and reads a line
    private native String getLine(String prompt);

    public static void main(String args[]) {
        Prompt p = new Prompt();
        String input = p.getLine("Type a line: ");
        System.out.println("User typed: " + input);
    }
    static {
        System.loadLibrary("Prompt");
    }
}
```

Prompt.main调用本地方法Prompt.getLine\(\)接收用户的输入。静态初始化块调用System.loadLibrary方法加载一个叫Prompt的本地库。
### 3.1.1 实现本地方法的C原型

Prompt.getLine方法可以用如下C函数实现：

```
JNIEXPORT jstring JNICALL
Java_Prompt_getLine(JNIEnv *env, jobject this, jstring prompt);
```

你可以使用javah命令\(第2章4节\)生成包含上述函数原型的头文件。JNIEXPORT和JNICALL宏（在jni.h头文件中定义）表明这个函数来自本地库。C函数的名字由“Java\_”前缀，类名和方法名组成。第11.3节中更准确的讲述了C函数名是如何生成的。

### 3.1.2 本地方法参数

正如第2.4节中讨论的，本地方法的实现例如Java\_Prompt\_getLine除了在本地方法声明的参数外还接收两个标准参数。第一个参数，JNIEnv接口指针，指向一个指向函数列表的指针地址。这个函数列表中的每个入口都指向一个JNI函数。本地方法总是通过Java虚拟机从其中一个JNI函数接收数据结构。图3.1描述了JNIEnv接口指针。  
![](/assets/Figure_3_1.PNG)  
图 3.1 JNIEnv接口指针  
第二个参数取决于本地方法是静态方法还是实例方法。如果本地方法是实例方法，第二个参数是一个对象的引用，类似于C++中的this指针。如果本地方法是静态方法，第二个参数是这个方法所在的类的引用。我们的例子中，Java\_Prompt\_getLine，实现一个实例本地方法。因此，jobject参数是一个对象本身的引用。

### 3.1.3 类型匹配

本地方法声明中的参数类型与本地程序语言中的类型相对应。JNI中定义了一套C和C++的类型与Java程序语言中的类型相对应。  
Java程序设计语言中有两个数据类型：基本数据类型如int,float,char和引用类型如类，实例以及数组。在Java中，字符串是java.lang.String类的实例。  
JNI对基本数据类型和引用类型的处理有所不同。基本数据类型的匹配相对简单。例如，Java中的int型相当于C\/C++中的jint类型（在jni.h中定义为32位有符号整型），Java中float则对应jfloat（在jni.h中定义为32位浮点数）。第12.1.1节包含了JNI中定义的所有基本数据类型。  
JNI将对象以不透明引用（opaque references）的方式传递给本地方法。不透明引用是C语言中的指针类型，涉及到Java虚拟机中的内部数据结构。真正的内部数据层是对程序员隐藏的。本地代码必须通过使用JNIEnv接口指针中的JNI函数来操作这些对象。例如，在JNI中与java.lang.String相对应的数据类型是jstring。而jstring引用的值与本地代码是不想管的。本地代码通过调用JNI中的函数GetStringUTFCars\($3.2.1\)去访问字符串的内容。  
所有JNI引用都有这类jobject。为了方便和安全，JNI定义了一套jobject的子类型。这些子类型对应Java程序设计中的常用引用类型。例如jstring对应字符串；jobjectArray对应数组队形。第12.1.2节包含了所有JNI的引用类型和他们相关的子类型。