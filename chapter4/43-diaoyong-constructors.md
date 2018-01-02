## 4.3 调用构造函数

在JNI中，构造函数可以按以下的步骤调用，类似调用实例方法。获取一个构造函数的方法ID，在方法描述符中传递“&lt;init&gt;”作为方法的名字，"V"作为返回类型。然后你可以通过传递方法ID给JNI函数调用构造函数，例如NewObject。以下代码实现JNI中的NewString函数的相同功能，把存储在C缓存的Unicode字符转换成java.lang.String对象。

```
jstring
MyNewString(JNIEnv *env, jchar *chars, jint len)
{
    jclass stringClass;
    jmethodID cid;
    jcharArray elemArr;
    jstring result;

    stringClass = (*env)->FindClass(env, "java/lang/String");
    if (stringClass == NULL) {
        return NULL; /* exception thrown */
    }
    /* Get the method ID for the String(char[]) constructor */
    cid = (*env)->GetMethodID(env, stringClass,
            "<init>", "([C)V");
    if (cid == NULL) {
        return NULL; /* exception thrown */
    }

    /* Create a char[] that holds the string characters */
    elemArr = (*env)->NewCharArray(env, len);
    if (elemArr == NULL) {
        return NULL; /* exception thrown */
    }
    (*env)->SetCharArrayRegion(env, elemArr, 0, len, chars);

    /* Construct a java.lang.String object */
    result = (*env)->NewObject(env, stringClass, cid, elemArr);

    /* Free local references */
    (*env)->DeleteLocalRef(env, elemArr);
    (*env)->DeleteLocalRef(env, stringClass);
    return result;
}
```

这个函数比较复杂，应该仔细的解释一下。首先，FindClass返回java.lang.String的引用。然后，GetMethodID返回字符串构造函数\(String\(char\[\] chars\)\)的方法ID。然后，我们调用NewCharArray函数动态分配一个字符数组保存所有字符串元素。NewObject函数将要构造的类的引用，构造函数的方法ID和参数传递给构造函数。

DeleteLocalRef调用允许虚拟机释放本地引用如elemArr和stringClass使用的资源。第5.2.1节将会详细描述什么时候，为什么你需要调用DeleteLocalRef。

字符串是对象。这个例子进一步强调了这一点。然而，这个例子也引出了一个问题。既然我们能够使用其他JNI函数实现相同的功能，为什么JNI还要提供内置函数例如NewString？因为调用内置函数比从本地代码调用java.lang.String的API更加高效。String是使用最频繁的对象类型，值得在JNI中提供特殊的支持。

也可以使用CallNonvirtualVoidMethod函数调用构造函数。这种情况下，本地代码必须先调用AllocObject函数创建一个未初始化的对象。上面调用的：

```
result = (*env)->NewObject(env, stringClass, cid, elemArr);
```

可以由如下代码代替：

```
result = (*env)->AllocObject(env, stringClass);
if (result) {
    (*env)->CallNonvirtualVoidMethod(env, result, stringClass,
    cid, elemArr);
    /* we need to check for possible exceptions */
    if ((*env)->ExceptionCheck(env)) {
        (*env)->DeleteLocalRef(env, result);
        result = NULL;
    }
}
```

AllocObject创建一个未初始化的对象，而且必须十分小心，确保每个对象中最多只调用构造函数一次。本地代码不许在同一个对象中重复调用构造函数。

有时候，你会发现先动态分配一个未初始化对象，然后某个时间以后调用构造函数是有用的。然而，在大多数情况下，你应该使用NewObject，并且避免使用容易出错的AllocObject/CallNonvirtualVoidMethod函数。

