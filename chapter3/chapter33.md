## 3.3 访问数组

JNI处理基本数组和对象数组的方式不同。基本数组包含的元素为基本数据类型，例如int和boolean。对象数组包含的元素为引用类型，例如类实例和其他数组。例如，下面用Java程序设计语言的代码段：

```
int[] iarr;
float[] farr;
Object[] oarr;
int[][] arr2;
```

iarr和farr是基本数组，而oarr和arr2是对象数组。

在本地方法中访问基本数组需要类似访问字符串的JNI函数。让我们看一下一个简单的例子。以下代码调用一个本地方法sumArray，将int数组中的值相加。

```
class IntArray {
    private native int sumArray(int[] arr);
    public static void main(String[] args) {
        IntArray p = new IntArray();
        int arr[] = new int[10];
        for (int i = 0; i < 10; i++) {
            arr[i] = i;
        }
        int sum = p.sumArray(arr);
        System.out.println("sum = " + sum);
    }
    static {
        System.loadLibrary("IntArray");
    }
}
```

### 3.3.1 C语言中访问数组

数组表示jarray引用类型，还有它的子类型例如jintArray。就像jstring不是一个C字符串类型，jarray也不是一个C数组类型。你不能通过jarray引用实现Java\_IntArray\_sumArray本地方法。以下C代码是非法的，并不会得到想要的结果：

```
/* This program is illegal! */
JNIEXPORT jint JNICALL
Java_IntArray_sumArray(JNIEnv *env, jobject obj, jintArray arr)
{
    int i, sum = 0;
    for (i = 0; i < 10; i++) {
        sum += arr[i];
    }
}
```

你必须使用相应的JNI函数来访问基本数组元素，如下所示：

```
JNIEXPORT jint JNICALL
Java_IntArray_sumArray(JNIEnv *env, jobject obj, jintArray arr)
{
    jint buf[10];
    jint i, sum = 0;
    (*env)->GetIntArrayRegion(env, arr, 0, 10, buf);
    for (i = 0; i < 10; i++) {
        sum += buf[i];
    }
    return sum;
}
```

### 3.3.2 访问基本类型数组

上一个例子使用GetIntArrayRegion函数将整形数组中的所有元素复制到C缓冲（buf）中。第三个参数是起始元素的索引，第四个参数是待拷贝元素的数目。当元素在C缓存中，我们就可以在本地代码中访问它们。不需要进行异常检查，因为我们知道在我们的例子中数组的长度为10，因此不会出现索引溢出。

JNI提供一个相应的SetIntArrayRegion函数允许本地代码修改int数组中的元素。同时也支持其他基本类型（例如boolean，short和float）的数组。

JNI提供一系列的Get/Release&lt;Type&gt;ArrayElements函数（例如Get/ReleaseIntArrayElements）允许本地代码获得指向基本数据类型数组中元素的指针。因为底层垃圾回收器也许不支持锁定，虚拟机也许会返回一个指向原始数组拷贝的指针。我们可以用GetIntArrayElements重新写第3.3.1章中的本地方法实现，如下：

```
JNIEXPORT jint JNICALL
Java_IntArray_sumArray(JNIEnv *env, jobject obj, jintArray arr)
{
    jint *carr;
    jint i, sum = 0;
    carr = (*env)->GetIntArrayElements(env, arr, NULL);
    if (carr == NULL) {
        return 0; /* exception occurred */
    }
    for (i=0; i<10; i++) {
        sum += carr[i];
    }
    (*env)->ReleaseIntArrayElements(env, arr, carr, 0);
    return sum;
}
```

GetArrayLength函数返回数组的元素个数。当数组第一次动态分配的时候，数组的长度就已经固定。

Java 2 SDK 1.2介绍了Get/ReleasePrimitiveArrayCritical函数。当本地代码访问基本数组的时候，这些函数允许虚拟机禁用垃圾回收。程序员需要像使用Get/ReleaseStringCritical函数一样注意。在一对Get/ReleasePrimitiveArrayCritical函数之间，本地代码不许调用任意的JNI函数，或者执行任何会造成应用程序死锁的阻塞操作。

### 3.3.3 JNI基本数组函数总结

下图是关于基本数据类型数组相关的JNI函数的总结。Java 2 SDK 1.2稳定版增加了许多新的函数来更好的操作数组。新增的函数除了提高性能之外，没有其他新的操作。![](/assets/Figure_3_4.PNG)3.3.4 在基本数组函数中选择

下图说明了一个程序员在JDK1.1和JDK1.2中如何选择JNI函数访问基本数组：![](/assets/Figure_3_5.PNG)如果你需要从C缓存拷贝内容或者拷贝内容到C缓存，请使用Set&lt;Type&gt;ArrayRegion族函数。这些函数执行边界检测并且当有必要的时候会处理ArrayIndexOutOfBoundsException异常。本地方法在第3.3.1节中的实现使用GetIntArrayRegion从jarray引用中复制出10个元素。

对于小的，固定长度的数组，Get/Set&lt;Type&gt;ArrayRegion函数总是比较常用，因为在C栈可以很廉价的分配C缓存。复制少量数组元素的开销可以忽略不计。

Get/Set&lt;Type&gt;ArrayRegion函数允许你指定一个起始索引和元素的数量，因此当只需要访问大型数组中的一个子集时，它是首选函数。

如果你没有预分配C缓存，那么原始数组的大小不确定，当指向数组元素的时候，本地代码不会发出阻塞调用，使用JDK1.2中的Get/ReleasePrimitiveArrayCritical函数。就像Get/ReleaseStringCritical函数，Get/ReleasePrimitiveArrayCritical函数在调用的时候必须格外小心，为了防止死锁。

使用Get/Release&lt;Type&gt;ArrayElements族函数总是安全的。虚拟机会返回一个直接指向数组元素的指针或者返回包含数组元素拷贝的缓存。

### 3.3.5 访问对象数组

JNI提供一组单独的函数访问对象数组。GetObjectArrayElement返回给定索引的元素，而SetObjectArrayElement更新给定索引的值。与基本数组类型不同，你不能获得所有的对象元素或者一次性复制多个对象元素。

字符串和数组是引用类型。你可以使用Get/SetObjectArrayElement访问字符串数组和数组的数组。

下面的例子调用一个本地方法创建一个二维整形数组，然后打印数组的内容。

```
class ObjectArrayTest {
    private static native int[][] initInt2DArray(int size);
    public static void main(String[] args) {
    int[][] i2arr = initInt2DArray(3);
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            System.out.print(" " + i2arr[i][j]);
        }
        System.out.println();
    }
}
    static {
        System.loadLibrary("ObjectArrayTest");
    }
}
```

静态本地方法initIntDArray创建一个给定大小的二维数组。本地方法动态分配和初始化一个二维数组可以写成如下：

```
JNIEXPORT jobjectArray JNICALL
Java_ObjectArrayTest_initInt2DArray(JNIEnv *env,
                                   jclass cls,
                                   int size)
{
    jobjectArray result;
    int i;
    jclass intArrCls = (*env)->FindClass(env, "[I");
    if (intArrCls == NULL) {
        return NULL; /* exception thrown */
    }
    result = (*env)->NewObjectArray(env, size, intArrCls,
                                    NULL);
    if (result == NULL) {
        return NULL; /* out of memory error thrown */
    }
    for (i = 0; i < size; i++) {
        jint tmp[256]; /* make sure it is large enough! */
        int j;
        jintArray iarr = (*env)->NewIntArray(env, size);
        if (iarr == NULL) {
            return NULL; /* out of memory error thrown */
        }
        for (j = 0; j < size; j++) {
            tmp[j] = i + j;
        }
        (*env)->SetIntArrayRegion(env, iarr, 0, size, tmp);
        (*env)->SetObjectArrayElement(env, result, i, iarr);
        (*env)->DeleteLocalRef(env, iarr);
    }
    return result;
}
```

newInt2DArray方法先调用JNI函数FindClass获取二维整型数组元素类的引用。FindClass的"\[I"参数是JNI的类描述符（$12.3.2）对应Java中的int\[\]类型。如果加载类失败（例如由于缺失类文件或内存不足），FindClass会返回NULL并且抛出一个异常。

接着NewObjectArray函数动态分配一个数组，其元素类型由intArrCls类引用表示。NewObjectArray函数只分配第一维，我们仍然需要构建第二维数组。Java虚拟机没有特别的数据结构支持多维数组。一个二维数组仅仅是一个数组的数组。

创建数组第二维度的代码十分简单。NewIntArray动态分配部分数组元素，SetIntArrayRegion复制tmp\[\]缓存中的内容到新分配的一维数组。 当SetObjectArrayElement完成调用以后，第i个一维数组的第j个元素的值为i+j。

运行ObjectArrayTest.main方法将会得到如下输出：

```
0 1 2
1 2 3
2 3 4
```

循环的末尾调用DeleteLocalRef是为了确保虚拟机不会耗尽内存，例如用来保存JNI引用的iarr。第5.2.1章将会详细解释什么时候并且为什么你需要调用DeleteLocalRef。



