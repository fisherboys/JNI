## 3.2 访问字符串

Java\_Prompt\_getLine函数接受jstring类型的参数。jstring类型代表Java虚拟机中的字符串，并且与C语言中的字符串类型不同（字符型指针，char \*）。你不能将jstring作为一个普通的C语言中的字符串使用。以下的代码是错误示例，运行也不会产生预期的结果。而且它会使Java虚拟机奔溃。

```
JNIEXPORT jstring JNICALL
Java_Prompt_getLine(JNIEnv *env, jobject obj, jstring prompt)
{
    /*错误：jstring的错误用法*/
    printf("%s", prompt);
    ...
}
```

### 3.2.1 转换成本地字符串

你的本地代码必须用相应的JNI函数将jstring对象转换成C\/C++中的字符串。JNI函数支持Unicaode和UTF-8字符串的相互转换。Unicode字符串将字符当做16位的值，而UTF-8字符串（$12.3.1）采用向上兼容7位ASCII字符串的编码方案。所有1到127之间的7位ASCII码与UTF-8的编码相同。  
Java\_Prompt\_getLine函数通过调用JNI函数GetStringUTFChars读取字符串的内容。GetStringUTFChars函数可以通过JNIEnv接口指针访问。它能够转换jstring引用类型，将Java虚拟机中的Unicode类型转换成C语言中的UTF-8类型。如果你确认原始字符串只包含了7位ASCII字符，你可以将字符串传递给C库函数，例如printf。（我们将在第8.2章介绍如何处理非ASCII字符串）

```
JNIEXPORT jstring JNICALL
Java_Prompt_getLine(JNIEnv *env, jobject obj, jstring prompt)
{
    char buf[128];
    const jbyte *str;
    str = (*env)->GetStringUTFChars(env, prompt, NULL);
    if (str == NULL) {
        return NULL;            //OutOfMemoryError already thrown
    }
    printf("%s", str);
    (*env)->ReleaseStringUTFChars(env, prompt, str);
    //假设用户不会输入超过127个字符
    scanf("%s", buf);
    return (*env)->NewStringUTF(env, buf);
}
```

别忘记验证GetStringUTFChars函数的返回值。因为Java虚拟机实现需要动态分配内存保存UTF-8字符串，有可能内存分配会失败。当这个问题发生时，GetStringUTFChars返回NULL并且扔出OutOfMemoryError异常。JNI中扔出一个异常与Java中的有所不同，我们将在第六章学习。在本地C语言中，JNI扔出的异常不会自动改变控制流。在C语言中，我们需要抛出一个明确的return状态来忽略保留的状态。当Java\_Prompt\_getLine返回时，异常会抛到调用Prompt.getLine本地方法的Prompt.main,

### 3.2.2 释放本地字符串资源

当你的本地代码不再使用GetStringUTFChars返回的UTF-8字符串的时候，它会调用ReleaseStringUTFChars。调用 ReleaseStringUTFChars 表明本地方法不需要再用到 GetStringUTFChars返回的UTF-8字符串。因此被UTF-8字符串占用的内存可以释放。调用ReleaseStringUTFChars失败将导致内存泄漏，最终导致内存耗尽。

### 3.2.3 建构新字符串

你可以在本地方法中调用JNI函数NewStringUTF创建一个新的java.lang.String引用。NewStringUTF函数接收一个UTF-8格式的C字符串，创建一个java.lang.String引用。

如果虚拟机不能为新创建的java.lang.String动态分配内存，NewStringUTF将抛出OutOfMemoryError并且返回NULL。在这个例子中，我们没必要检查它的返回值因为本地方法之后立即返回。如果NewStringUTF失败，OutOfMemoryError异常将抛出到Prompt.main。如果NewStringUTF成功调用，它将返回JNI引用给java.lang.String的实例。这个新的实例由Prompt.getLine返回，并作为本地变量赋值给Promt.main输入。

### 3.2.4 其他JNI字符串函数

除了前面介绍的GetStringUTFChars， ReleaseStringUTFChars和NewStringUTF函数之外，JNI还提供了其他字符串相关的函数。

GetStringUTFChars和ReleaseStringUTFChars获取Unicode格式的字符。当操作系统支持Unicode编码的时候，这些函数非常有用。

UTF-8字符串通常以'\0'结尾，而Unicode不是。想知道一个jstring引用里面Unicode字符的数量，JNI程序员可以调用GetStringLength函数。如果想知道一个UTF-8格式的jstring的长度，则可先调用GetStringUTFChars得到值，再调用C函数中的strlen，或者直接调用JNI函数中的GetStringUTFLength。

GetStringUTFChars和GetStringChars的第三个参数需要额外解释一下：

```
const jchar *
GetStringChars(JNIEnv *env, jstring str, jboolean *isCopy);
```

当GetStringChars返回值的时候，如果返回的字符串是原始字符串的一份拷贝，则指向isCopy的内存地址的值将设置为JNI\_TRUE。如果返回的字符串是原来的java.lang.String引用，则指向isCopy的内存地址的值将设置为JNI\_FALSE。 如果isCopy所指向的地址被设置成JNI\_FALSE，则本地代码不许修改返回字符串的值。如果违反这个规则，将导致原始字符串java.lang.String的引用也被修改。这将导致打破常量是不可变的规则。

大多数情况下，你将isCopy的参数设置为NULL，因为你不在乎Java虚拟机返回的是一个字符串的拷贝还是字符串它本身。

一般的，我们无法预测虚拟机是否会拷贝给定的字符串。因此程序员必须假设类似GetStringChars的函数可能花费一定比例的时间和空间在java.lang.String引用的数量上。一个典型的Java虚拟机实例中，垃圾回收器回收堆中的对象。一旦直接指向java.lang.String引用的指针传回给本地代码，垃圾回收器将不再回收这个java.lang.String引用。换句话说，虚拟机pin这个java.lang.String引用。因为过多的pin将导致内存碎片，虚拟机将自己权衡决定是赋值字符串还是pin引用。

当你不再需要访问一个GetStringChars返回的字符串以后，不要忘了调用ReleaseStringChars。不论GetStringChars函数中的\*isCopy是JNI\_TRUE还是JNI\_FALSE，调用ReleaseStringChars都是有必要的。取决于GetStringChars返回真还是假，ReleaseStringChars都可以释放相应的资源。

### 3.2.5 Java 2 SDK Release 1.2中新增的JNI字符串相关函数

为了让虚拟机更有可能返回直接指向字符串的指针，Java 2 SDK release1.2 新增了一对函数：Get\/ReleaseStringCritical。从表面上看，他们类似于Get\/ReleaseStringChars函数。但是使用这些函数有很多限制。

你必须把这对函数里的代码看作一个临界区域（critical region）。在这个区域内，本地代码不准调用任意的JNI函数或者任何会造成当前线程堵塞和等待运行在Java虚拟机中的线程的函数。例如，当前线程不准等待另一个线程的I\/O流输入。

当本地代码通过GetStringCritical直接指向一个字符串元素的时候，这些限制使得虚拟机禁止垃圾回收。当垃圾回收被禁止，任何触发垃圾回收的线程也会被禁止。在Java虚拟机中，包含Get\/ReleaseStringCritical的本地代码不许阻塞调用或者分配新的对象。否则，虚拟机将被锁死。考虑下面的场景：

* 由另一个线程触发的垃圾回收不能执行直到当前线程结束阻塞调用并且重新使能垃圾回收。
* 于此同时，当前线程不能执行，因为阻塞调用需要遵循由另一个线程锁。

重复使用多对GetStringCritical和ReleaseStringCritical函数式安全的。例如：

```
jchar *s1, *s2;
s1 = (*env)->GetStringCritical(env, jstr1);
if (s1 == NULL)
{
    .../* error handling */
}
s2 = (*env)->GetStringCritical(env, jstr2);
if (s2 == NULL) {
    (*env)->ReleaseStringCritical(env, jstr1, s1);
    .../*error handling*/
}
...//use s1 and s2
(*env)->ReleaseStringCritical(env, jstr1, s1);
(*env)->ReleaseStringCritical(env, jstr2, s2);
```

在一个栈中，Get\/ReleaseStringCritical不必严格的挨着。我们不能忘记检查返回值是否为NULL，因为 ，如果虚拟机内部的数组为另一种格式，GetStringCritical或许仍然动态分配一部分空间并且复制一个数组。例如，Java虚拟机中的数组不是连续存储的。这种情况，GetStringCritical必须复制jstring实例中的所有字符为了返回一个联系的字符数组给本地代码。

为了防止死锁，你必须确保，在执行GetStringCritical之后，相应的ReleaseStringCritical调用之前，本地代码不会调用任意的JNI函数。只有Get\/ReleaseStringCritical和Get\/ReleasePrimitiveArrayCritical才可以在这个区域内调用。JNI不支持GetStringUTFCritical和ReleaseStringUTFCritical函数。这类函数需要虚拟机拷贝一份字符串，因为虚拟机的实现几乎肯定内部字符串为Unicode格式。

另外还有GetStringRegion和GetStringUTFRegion。这些函数将字符串元素复制给预先分配的空间。Prompt.getLine方法可以用GetStringUTFRegion重新实现：

```
JNIEXPORT jstring JNICALL
Java_Prompt_getLine(JNIEnv *env, jobject obj, jstring prompt)
{
    /* assume the prompt string and user input has less than 128 characters */
    char outbuf[128], inbuf[128];
    int len = (*env)->GetStringLength(env, prompt);
    (*env)->GetStringUTFRegion(env, prompt, 0, len, outbuf);
    printf("%s", outbuf);
    scanf("%s", inbuf);
    return (*env)->NewStringUTF(env, inbuf);
}
```

GetStringUTFRegion函数取一个开始的索引和长度，对Unicode字符进行计数。这个函数需要进行边界检查，如果有必要则要抛出StringIndexOutOfBoundsException异常。在上面的代码中，我们遵循字符引用它本身的长度，所以确保没有指数溢出。（然后上面的代码缺少必要的检查，保证字符串少于128个字符）

这个代码比用GetStringUTFChars稍微简单一些。因为GetStringUTFRegion没有动态内存分配，我们不必检查内存溢出的情况。

### 3.2.6 JNI字符串函数的总结

图3.2总结了所有字符串相关的JNI函数。Java 2 SDK 1.2稳定版增加了许多新的函数来更好的操作字符串。新增的函数除了提高性能之外，没有其他新的操作。

### ![](/assets/Figure_3_2.PNG)

### 3.2.7 在字符串函数中选择

图3.3描述了在JDK1.1和JDK1.2中，一个程序员如何在字符串相关函数中选择。

![](/assets/Figure_3_3.PNG)

如果你的JNI目标版本是1.1或者1.1和1.2，你只能使用Get/ReleaseStringChars和Get/ReleaseStringUTFChars。

如果你使用1.2版本或者更高的版本，同时你想将字符串的内容复制给已经分配内存的C，那么用GetStringRegion或者GetStringUTFRegion。

对于固定长度的字符串，Get/SetStringRegion和Get/SetStringUTFRegion一般更适合，因为C内存在C的栈中很廉价。复制一个字符串的花费不值一提。

Get/SetStringRegion和Get\/SetStringUTFRegion的一个优点是他们不进行内存动态分配，因为不会发生无法预知的内存溢出的异常。如果你确保指数溢出不会发生，异常检测不是必须的。

Get/SetStringRegion和Get\/SetStringUTFRegion的另一个优点是你可以指定开始索引和字符的数量。如果本地代码仅仅需要访问一个很长字符串的子串，这些函数很合适。

GetStringCritical用起来必须格外的小心（$3.2.5）。你必须保证当通过GetStringCritical获得一个指针的时候，本地代码不会再Java虚拟机中动态分配新的对象或者执行可能造成系统死锁的阻塞调用。

以下是一个演示使用GetStringCritical造成微妙问题的例子。以下代码从一个字符串获得内容，并调用fprintf函数将字符写出到文件句柄fd：

```
/* This is not safe! */
const char *c_str = (*env)->GetStringCritical(env, j_str, 0);
if (c_str == NULL) {
    ... /* error handling */
}
fprintf(fd, "%s\n", c_str);
(*env)->ReleaseStringCritical(env, j_str, c_str);
```

以上代码的问题是当当前线程禁用垃圾回收的时候，写入文件句柄的操作不总是安全的。例如，假设另一个线程T正等待读取fd文件句柄。我们进一步假设操作系统缓冲建立一种方式，fprintf调用等待直到线程T完成从fd读取所有等待数据。我们构建一个死锁的可能场景：如果线程T不能动态分配足够的内存空间作为提供给读取文件句柄的缓存，它必须申请垃圾回收。垃圾回收的请求将被阻塞直到fprintf调用返回。然而，fprintf调用在等待线程T完成从文件句柄读取。

以下代码，虽然跟之前的例子相似，但不会死锁：

```
/* This code segment is OK. */
const char *c_str = (*env)->GetStringCritical(env, j_str, 0);
if (c_str == NULL) {
    ... /* error handling */
}
DrawString(c_str);
(*env)->ReleaseStringCritical(env, j_str, c_str);
```

DrawString是直接将字符串输出到屏幕的系统调用。除非显示屏的驱动也是由同一个虚拟机中的Java应用程序驱动的，否则DrawString函数不会无限期阻塞垃圾回收的发生。