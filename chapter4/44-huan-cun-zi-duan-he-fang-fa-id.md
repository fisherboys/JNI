## 4.4 缓存字段和方法ID

获取字段和方法ID时，需要根据字段或者方法的名字和描述符查找。查找相对来说比较费时。这一节，我们将介绍一种降低开销的技术。

思路是计算字段和方法ID，并且缓存起来以备之后使用。目前有两种方式缓存字段和方法ID，取决于缓存发生的时刻，是在使用字段和方法ID时，还是在字段和方法定义所在类的静态初始化时。

### 4.4.1 在使用时缓存

字段和方法ID会在本地代码调用字段值或者执行方法回调的时候缓存。Java\_InstanceFieldAccess\_accessField函数的以下实现将字段ID缓存在静态变量中，所以当InstanceFieldAccess.accessField方法每次调用的时候，它不必重新计算。

```
JNIEXPORT void JNICALL
Java_InstanceFieldAccess_accessField(JNIEnv *env, jobject obj)
{
    static jfieldID fid_s = NULL; /* cached field ID for s */
    
    jclass cls = (*env)->GetObjectClass(env, obj);
    jstring jstr;
    const char *str;
    
    if (fid_s == NULL) {
        fid_s = (*env)->GetFieldID(env, cls, "s",
                                    "Ljava/lang/String;");
        if (fid_s == NULL) {
            return; /* exception already thrown */
        }
    }
    
    printf("In C:\n");
    
    jstr = (*env)->GetObjectField(env, obj, fid_s);
    str = (*env)->GetStringUTFChars(env, jstr, NULL);
    if (str == NULL) {
        return; /* out of memory */
    }
    printf(" c.s = \"%s\"\n", str);
    (*env)->ReleaseStringUTFChars(env, jstr, str);
    jstr = (*env)->NewStringUTF(env, "123");
    if (jstr == NULL) {
        return; /* out of memory */
    }
    (*env)->SetObjectField(env, obj, fid_s, jstr);
}
```

高亮的静态变量fid\_s存储预先计算的字段ID。这个静态变量初始为NULL。当InstanceFieldAccess.accessField方法第一次调用的时候，计算得到字段ID并且缓存在这个静态变量当中以备之后使用。

你可能已经注意到在上面的代码中存在着明显的竞争条件。多线程可能同时调用InstanceFieldAccess.accessField方法，并同时计算字段ID。一个线程可能覆写另一个线程计算得到的静态变量fid\_s。幸运的是，虽然这个竞争条件导致了多线程中的重复工作，然而这没事。同一个类中，由多线程计算得到的字段都是相同的。

遵循同样的想法，我们也可以缓存之前MyBewString例子中的java.lang.String构造方法的ID。

```
jstring
MyNewString(JNIEnv *env, jchar *chars, jint len)
{
    jclass stringClass;
    jcharArray elemArr;
    static jmethodID cid = NULL;
    jstring result;
    
    stringClass = (*env)->FindClass(env, "java/lang/String");
    if (stringClass == NULL) {
        return NULL; /* exception thrown */
    }
    
    /* Note that cid is a static variable */
    if (cid == NULL) {
    /* Get the method ID for the String constructor */
        cid = (*env)->GetMethodID(env, stringClass,
                                    "<init>", "([C)V");
        if (cid == NULL) {
            return NULL; /* exception thrown */
        }
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

当第一次调用MyNewString的时候，我们计算java.lang.String构造方法的方法ID。高亮的静态变量**cid**缓存计算结果。

### 4.4.2 



