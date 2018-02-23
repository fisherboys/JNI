## 5.1.2 全局引用

你可以在多个调用本地方法的时候使用全局引用。全局引用可以在多线程之间使用且有效，直到程序员手动释放它。跟局部引用一样，全局引用确保被引用的对象不会被GC回收。

与局部引用可以被大多数JNI创建不同的是，全局引用只能由一个JNI函数创建：NewGlobalRef。以下版本的MyNewString举例说明如何使用一个全局引用。我们高亮以下代码和上一节中错误的缓存局部引用的代码的不同：

```
/* This code is OK */
jstring
MyNewString(JNIEnv *env, jchar *chars, jint len)
{
    static jclass stringClass = NULL;
    ...
    if (stringClass == NULL) {
        jclass localRefCls =
                    (*env)->FindClass(env, "java/lang/String");
        if (localRefCls == NULL) {
            return NULL; /* exception thrown */
        }
        //以下代码高亮
        /* Create a global reference */
        stringClass = (*env)->NewGlobalRef(env, localRefCls);
    
        /* The local reference is no longer useful */
        (*env)->DeleteLocalRef(env, localRefCls);
    
        /* Is the global reference created successfully? */
        if (stringClass == NULL) {
            return NULL; /* out of memory exception thrown */
        }
        //高亮end
    }
    ...
}
```

修改后的版本将FindClass返回的局部引用传递给NewGlobalRef，用来创建一个对String类的全局引用。在删除localRefCls以后，我们检查NewGlobalRef是否成功创建了stringClass，因为不论发生何种情况局部引用需要被删除。

