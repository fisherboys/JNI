## 4.1 访问域

Java程序设计语言支持两种类型的域。每个类的实例拥有类的实例域的自身副本，而这个类的所有实例共用这个类的静态域。

JNI提供函数使本地代码能获得和设置对象中的实例域和类中的静态域。我们首先看看一个例子描述如何从一个本地方法实现访问实例域。

```
class InstanceFieldAccess {
    private String s;
    
    private native void accessField();
    public static void main(String args[]) {
        InstanceFieldAccess c = new InstanceFieldAccess();
        c.s = "abc";
        c.accessField();
        System.out.println("In Java:");
        System.out.println(" c.s = \"" + c.s + "\"");
    }
    static {
        System.loadLibrary("InstanceFieldAccess");
    }
}
```

InstanceFieldAccess类定义一个实例域s。主函数创建一个对象，设置实例域，接着调用本地方法InstanceFieldAccess.accessField。正如我们很快看到，本地方法打印实例域的已有值，接着将域设置一个新值。当本地方法返回后，程序再次打印域的值，表明域值确实改变了。

以下是InstanceFieldAccess.accessField本地方法的具体实现。

```
JNIEXPORT void JNICALL
Java_InstanceFieldAccess_accessField(JNIEnv *env, jobject obj)
{
    jfieldID fid; /* store the field ID */
    jstring jstr;
    const char *str;
    /* Get a reference to obj’s class */
    jclass cls = (*env)->GetObjectClass(env, obj);
    printf("In C:\n");
    /* Look for the instance field s in cls */
    fid = (*env)->GetFieldID(env, cls, "s",
                                "Ljava/lang/String;");
    if (fid == NULL) {
        return; /* failed to find the field */
    }
    /* Read the instance field s */
    jstr = (*env)->GetObjectField(env, obj, fid);
    str = (*env)->GetStringUTFChars(env, jstr, NULL);
    if (str == NULL) {
        return; /* out of memory */
    }
    printf(" c.s = \"%s\"\n", str);
    (*env)->ReleaseStringUTFChars(env, jstr, str);
    /* Create a new string and overwrite the instance field */
    jstr = (*env)->NewStringUTF(env, "123");
    if (jstr == NULL) {
        return; /* out of memory */
    }
    (*env)->SetObjectField(env, obj, fid, jstr);
}
```

运行InstanceFieldAccess类和InstanceFieldAccess本地库得到如下输出：

```
In C:
    c.s = "abc"

In Java:
    c.s = "123"
```



