因为本书涉及的例子即使用了Java语言也使用了本地语言（C, C++等），让我们先弄清楚这些语言编译环境的关联。

Java平台是由Java虚拟机\(VM\)和Java应用程序接口\(API\)组成的编译环境。Java应用程序由Java编译语言编写，然后编译成机器识别的二进制类格式。类可以在任意Java虚拟机上运行。Java API由一系列预先定义好的类组成。任何装有Java平台的设备都可以支持Java程序设计语言、虚拟机和API。

术语“宿主环境”代表主机的运行系统、本地库和CPU指令集。本地应用程序（native applications）由本地程序设计语言（native programming languages）编写，例如C和C++，编译成宿主特定的二进制码，并与本地库相连接。本地应用程序和本地库通常依赖于某个宿主环境。例如一个C应用程序能在一个系统平台上运行，则通常不能再另一个系统平台上运行。

Java平台通常部署在宿主环境之上。例如Java运行时\(Java Runtime Environment, JRE\)是Sun公司的一个产品，用来支持Java平台运行在现有运行系统如Solaris和Windows之上。Java平台提供一套特性使应用程序独立于下层的宿主环境。