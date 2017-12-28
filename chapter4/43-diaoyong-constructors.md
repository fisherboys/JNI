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



