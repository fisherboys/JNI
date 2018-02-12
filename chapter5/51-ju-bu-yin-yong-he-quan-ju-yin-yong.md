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

这里，我们省略了和我们讨论不相关的代码。

