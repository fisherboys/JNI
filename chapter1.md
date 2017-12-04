Java本地接口（JNI）是Java语言的一个强大功能。使用JNI的应用可以包含本地代码（native code），如C和C++。JNI使得程序员能够充分利用Java语言的强大而不用放弃之前所用的代码。由于JNI是Java语言的一部分，所以程序员仅仅需要解决一次问题，就可以在其他Java的扩展程序中应用。

这本书即是一本编程指南也是一本JNI的参考手册。它由三部分组成：

* 第二章通过一个简单的例子介绍JNI，为了让初学者熟悉JNI。
* 第三章到第十章则相当于编程指南，简要概述了JNI的特性。我们会通过一系列短小但具有描述性的例子去突出各个JNI特性，并且分享一些被证实在JNI编程中实用的技术。
* 第十一章到第十三章讲述JNI类型和函数的规范。这些章节同时充当了参考书的角色。

这本书试着去满足对JNI存在不同诉求的用户。使用说明书和编程指南主要针对初级开发人员，而有经验的开发者和JNI专家则会发现参考手册更实用。大部分的读者可能是使用JNI写应用程序的开发人员。书中提到的“你”暗指使用JNI编程的开发者，而不是JNI 专家或者使用JNI写的应用程序的用户。

本书默认你具备Java，C和C++编程语言的基础知识。如果你不具备，你最好参考众多经典中的一本：Ken Arnold和James Gosling的The Java Programming Launguage、Brian Kernighan和Dennis Ritchie的The C Programming Launguage、还有Bjarne Stroustrup的The C++ Programming Launguage。

本章的剩余部分将介绍JNI的背景、作用和发展。

