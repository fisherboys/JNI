## 5.1.3 弱全局引用

弱全局引用是Java 2 SDK稳定版1.2的新特性。他们通过NewGlobalWeakRef创建，由DeleteGlobalWeakRef释放。与全局引用一样，弱全局引用可以跨本地方法、跨线程使用。与全局引用不同的是，弱全局引用不会阻止GC回收相应的对象。

MyNewString例子已经展示如何缓存一个指向java.lang.String类的全局引用。MyNewString例子也可以使用弱全局引用缓存java.lang.String类。我们是使用全局引用还是弱全局引用都没关系，因为java.lang.String是一个系统类，它不会被GC回收。

当本地代码中缓存的引用不一定要阻止GC回收相应对象的时候，弱全局引用会变得更加有效。例如，假设一个本地方法mypkg.MyCls.f需要缓存类mypkg.MyCls2的引用。以弱全局引用的方式缓存类仍然可以让mypkg.MyCls2类卸载（unloaded）：

```
JNIEXPORT void JNICALL
Java_mypkg_MyCls_f(JNIEnv *env, jobject self)
{
    static jclass myCls2 = NULL;
    if (myCls2 == NULL) {
        jclass myCls2Local =
            (*env)->FindClass(env, "mypkg/MyCls2");
        if (myCls2Local == NULL) {
            return; /* can’t find class */
        }
        myCls2 = NewWeakGlobalRef(env, myCls2Local);
        if (myCls2 == NULL) {
            return; /* out of memory */
        }
    }
    ... /* use myCls2 */
}
```

我们假设MyCls和MyCls2有相同的声明周期。（例如，他们被相同的类加载器加载。）然而我们不想出现这种情况，当MyCls2已经被卸载，随后又被加载。如果这种情况发生，我们需要判断缓存的弱全局引用是指向一个活动的类对象还是指向一个已经被GC回收的类对象。下一节将告诉你怎么检查弱全局引用。

