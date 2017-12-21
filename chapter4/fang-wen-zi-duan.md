## 4.1 访问变量

Java程序设计语言支持两种类型的变量。每个类的实例拥有类的实例变量的自身副本，和这个类的所有实例共用这个类的静态变量。

JNI提供函数使本地代码能获得和设置对象中的实例变量和类中的静态变量。我们首先看看一个例子描述如何从一个本地方法实现访问实例变量（普通变量）。

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

InstanceFieldAccess类定义一个变量s。主函数创建一个对象，设置变量值，接着调用本地方法InstanceFieldAccess.accessField。正如我们很快看到，本地方法打印该变量的已有值，接着将变量设置一个新值。当本地方法返回后，程序再次打印变量的值，表明变量值确实改变了。

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

### 4.1.1 访问变量的流程

为了访问变量，本地方法遵循两个步骤。首先，调用GetFieldID从类引用中获得变量的ID，变量名和域描述符：

```
fid = (*env)->GetFieldID(env, cls, "s", "Ljava/lang/String;");
```

示例代码通过调用obj实例的GetObjectClass函数得到类引用cls，cls作为第二个参数传递给本地方法。

当你获取了变量的ID，你能够传递对象引用和变量ID访问相应函数：

```
jstr = (*env)->GetObjectField(env, obj, fid);
```

因为字符串和数组是特殊的对象，我们需要使用GetObjectField去访问字符串的变量。除了Get/SetObjectField,JNI也提供其他的函数如GetIntField和SetFloatField来访问基本数据类型的变量。

### 4.1.2 变量描述符

在之前的小节中，你也许已经注意到我们用到了一个特别的C字符串“Ljava/lang/String;”代表变量的类型。这些C字符串叫做JNI域描述符。

字符串的内容由声明变量的类型决定。例如，我们用"I"代表整型变量，“F代表浮点型变量，“D”代表双整型，“Z”代表布尔型等等。

引用类型的描述符，例如java.lang.String，以字母L开头，紧跟JNI类描述符\($3.3.5\),以分号结束。完整类名中的"."分隔符在JNI类描述符中替换为"/"。因此，java.lang.String类型的变量描述符为：

```
“Ljava/lang/String;”
```

数组类型的描述符由"\["开头，随后是数组元素描述符。例如"\[I"是int\[\]域的描述符。第12.3.3节介绍了变量描述符的细节和他们对象Java中的类型。

你可以使用javap工具通过类文件生成描述符。通常javap打印出给定类中方法和变量类型。如果你添加-s选项\(-p选项针对私有成员\)，javap打印JNI描述符：

```
javap -s -p InstanceFieldAccess
```

这将会包含变量s的JNI描述符的输出：

```
...
s Ljava/lang/String;
...
```

使用javap工具有助于消除由人工得到字符串的JNI描述符的错误。

### 4.1.3 访问静态变量

访问静态变量类似于访问实例变量。我们先看下InstanceFieldAccess例子的小的变化：

```
class StaticFielcdAccess {
    private static int si;

    private native void accessField();
    public static void main(String args[]) {
        StaticFieldAccess c = new StaticFieldAccess();
        StaticFieldAccess.si = 100;
        c.accessField();
        System.out.println("In Java:");
        System.out.println(" StaticFieldAccess.si = " + si);
    }
    static {
        System.loadLibrary("StaticFieldAccess");
    }
}
```

StaticFieldAccess类包含一个静态整型域si。StaticFieldAccess.main方法创建一个对象，初始化静态变量，然后调用本地方法StaticFieldAccess.accessField。正如我们随后看到的，本地方法打印已有静态变量的值，随后将该变量设置一个新的值。为了证明该变量的确被改变了，当本地方法调用以后，程序再次打印静态变量的值。

以下是StaticFieldAccess.accessField本地方法的实现。

```
JNIEXPORT void JNICALL
Java_StaticFieldAccess_accessField(JNIEnv *env, jobject obj)
{
    jfieldID fid; /* store the field ID */
    jint si;
    /* Get a reference to obj’s class */
    jclass cls = (*env)->GetObjectClass(env, obj);
    printf("In C:\n");
    /* Look for the static field si in cls */
    fid = (*env)->GetStaticFieldID(env, cls, "si", "I");
    if (fid == NULL) {
        return; /* field not found */
    }
    /* Access the static field si */
    si = (*env)->GetStaticIntField(env, cls, fid);
    printf(" StaticFieldAccess.si = %d\n", si);
    (*env)->SetStaticIntField(env, cls, fid, 200);
}
```

运行程序得到如下输出：

```
In C:
    StaticFieldAccess.si = 100
In Java:
    StaticFieldAccess.si = 200
```

访问静态变量和访问实例变量有两个不同点：

1. 对于静态变量，需要调用GetStaticFieldID,而对于实例变量，需要调用GetFieldID。GetStaticFieldID和GetFieldID有相同的返回类型jfieldID；
2. 一旦你获得了静态变量的ID，传递类引用给相应的静态变量访问函数，相对于对象引用。



