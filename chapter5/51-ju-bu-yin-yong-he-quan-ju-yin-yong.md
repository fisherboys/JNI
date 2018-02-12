## 5.1 局部引用和全局引用

什么是局部引用和全局引用，他们有什么不同？我们将用一些列的例子阐明局部引用和全局引用。

### 5.1.1 局部引用

大多数JNI函数创建局部引用。例如，JNI函数NewObject创建一个新的实例，并且返回一个实例的局部引用。

一个局部引用只有在创建它的本地方法返回前有效。一旦本地方法返回，所有本地方法执行期间创建的局部引用都会被释放。

你不能在本地方法中把局部引用储存在静态变量中，然后在随后的调用中使用这个引用。例如，以下代码错误是根据4.4.1中MyNewString函数修改的，是使用局部引用的**错误示例**：

```
/* This code is illegal */
jstring
MyNewString(JNIEnv *env, jchar *chars, jint len)
{
    static jclass stringClass = NULL;
    jmethodID cid;
    jcharArray elemArr;
    jstring result;

    if (stringClass == NULL) {
        stringClass = (*env)->FindClass(env,
                            "java/lang/String");
        if (stringClass == NULL) {
            return NULL; /* exception thrown */
        }
    }
    /* It is wrong to use the cached stringClass here,
        because it may be invalid. */
    cid = (*env)->GetMethodID(env, stringClass,
                                "<init>", "([C)V");
    ...
    elemArr = (*env)->NewCharArray(env, len);
    ...
    result = (*env)->NewObject(env, stringClass, cid, elemArr);
    (*env)->DeleteLocalRef(env, elemArr);
    return result;
}
```

这里，我们省略了和我们讨论不相关的代码。将stringClass缓存在一个静态变量是为了消除重复调用以下函数而造成的开销：

```
FindClass(env, "java/lang/String");
```

这种方式是不正确的，因为FindClass返回一个java.lang.String的局部引用。为了明白为什么这是个问题，假设一个本地方法C.f调用了MyNewString：

```
JNIEXPORT jstring JNICALL
Java_C_f(JNIEnv *env, jobject this)
{
    char *c_str = ...;
    ...
    return MyNewString(c_str);
}
```

当本地方法C.f返回以后，虚拟机释放Java\_C\_f执行期间的所有局部引用。这些释放的局部引用包括类对象保存的stringClass变量的局部引用。将来再次调用MyNewString时，会试图访问一个无效的局部引用，会导致非法的内存访问或者系统奔溃。如下代码段两次连续调用C.f，导致MyNewString指向无效的局部引用：

```
...
... = C.f(); // The first call is perhaps OK.
... = C.f(); // This would use an invalid local reference.
...
```

有两种方式可以释放局部引用。如之前解释，虚拟机会在本地方法返回时，自动释放执行本地方法期间所创建的所有局部引用。另外，程序员可以使用JNI函数DeleteLocalRef手动释放局部引用。

既然虚拟机会在本地方法结束时自动释放局部引用，我们为什么还要手动释放呢？一个局部引用会防止引用对象被GC回收，直到局部引用无效。DeleteKocalRef可以迅速将数组对象（elemArr）迅速回收。然而，虚拟机只会在本地方法调用MyNewString（如C.f）时才会释放elemArr对象。

一个局部引用在销毁之前也许会在多个本地方法之间传递。例如，MyNewString返回一个字符串引用。将由MyNewString的调用者决定是否释放MyNewString返回的局部引用。在Java\_C\_f例子中，C.f依次返回MyNewString的结果。当虚拟机收到从Java\_C\_f的局部引用，它将字符串对象传给C.f的调用者，然后销毁不低引用。

局部引用只在创建它们的线程中有效。一个线程中创建的局部引用不能在另一个线程中使用。不要在一个线程中创建局部引用并存储到全局引用中，然后到另外一个线程去使用。

