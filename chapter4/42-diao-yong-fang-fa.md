## 4.2 调用方法

Java中有几种方法。实例方法必须在类的特定实例上调用，然而静态方法可以独立于任何实例调用。我们将构造函数的讨论推迟到下一节。

JNI提供完整的一套函数允许你从本地代码执行回调。下面的示例代码包括一个本地方法调用Java中实现的实例方法。

```
class InstanceMethodCall {
    private native void nativeMethod();
    private void callback() {
        System.out.println("In Java");
    }
    public static void main(String args[]) {
        InstanceMethodCall c = new InstanceMethodCall();
        c.nativeMethod();
    }
    static {
        System.loadLibrary("InstanceMethodCall");
    }
}
```

下面是本地方法的实现：

```
JNIEXPORT void JNICALL
Java_InstanceMethodCall_nativeMethod(JNIEnv *env, jobject obj)
{
    jclass cls = (*env)->GetObjectClass(env, obj);
    jmethodID mid =
        (*env)->GetMethodID(env, cls, "callback", "()V");
    if (mid == NULL) {
        return; /* method not found */
    }
    printf("In C\n");
    (*env)->CallVoidMethod(env, obj, mid);
}
```

运行以上代码得到如下输出：

```
In C
In Java
```

### 4.2.1 调用实例方法

Java\_InstanceMethodCall\_nativeMethod实现表明调用实例方法需要两个步骤：

* 首先，本地方法调用JNI函数GetMethodID. GetMethodID在给定的类中查询方法。查询基于方法的名字和类型描述符。如果方法不存在，GetMethodID返回空。这时候，本地方法的立即返回会在调用InstanceMethodCall.nativeMethod的代码中抛出NoSuchMethodError异常。
* 然后，本地方法调用CallVoidMethod.CallVoidMethod调用返回为空的实例方法。传递对象，方法ID和实际变量（上述例子中没有）给CallVoidMethod方法。

除了CallVoidMethod函数，JNI也支持其他返回值的方法调用函数。例如，如果你调用的方法返回int类型，那么你的本地方法需要使用CallIntMethod. 类似的，你可以使用CallObejectMethod去调用返回对象类型的方法，包括java.lang.String实例和数组。

你也可以使用Call&lt;Type&gt;Method族函数调用接口方法。必须从接口类型派生方法ID。例如，以下代码片段调用java.lang.Thread实例中的Runnable.run方法：

```
jobject thd = ...; /* a java.lang.Thread instance */
jmethodID mid;
jclass runnableIntf =
    (*env)->FindClass(env, "java/lang/Runnable");
if (runnableIntf == NULL) {
    ... /* error handling */
}
mid = (*env)->GetMethodID(env, runnableIntf, "run", "()V");
if (mid == NULL) {
    ... /* error handling */
}
(*env)->CallVoidMethod(env, thd, mid);
... /* check for possible exceptions */
```

我们已经在第3.3.5节中讨论过FindClass函数返回一个引用给指定名字的类。同样在这，我们用它获得一个引用给指定的接口。

### 4.2.2 形成方法描述符

JNI使用描述符字符串代表方法类型，类似其代表变量类型一样。一个方法描述符结合参数类型和方法的返回类型。变量类型在前，并由小括号括起来。变量类型按照它们在声明方法时的顺序排列。多个变量类型之间没有分隔符。如果这个方法没有变量，那么就由一对空的小括号代替。方法的返回类型紧跟小括号的右边。

例如，"\(I\)V"代表这个方法有一个int类型的变量，并返回空。"\(\)D"代表这个方法没有变量，返回一个double类型的值。不要让C函数原型，例如"int f\(void\)"，误导你认为"\(V\)I"是一个有效的方法描述符，应该用"\(\)I"代替。

方法描述符也许会包括类描述符\($12.3.2\)。例如，方法：

```
native private String getLine(String);
```

有以下描述符：

```
"(Ljava/lang/String;)Ljava/lang/String;"
```

数组的描述符以"\["开头，紧接着数组元素类型的描述符。例如，以下方法：

```
public static void main(String[] args)
```

它的描述符为

```
"([Ljava/lang/String;)V"
```

第12.3.4章节对如何形成JNI方法描述符做了完整描述。你可以使用javap工具得到JNI方法描述符。例如，执行：

```
javap -s -p InstanceMethodCall
```

你可以得到如下输出：

```
...
private callback ()V
public static main ([Ljava/lang/String;)V
private native nativeMethod ()V
...
```

-s参数告诉javap输出JNI描述符字符串，而不是正如它们在Java中出现的类型。-p参数让javap输出的信息包含类的私有成员。



