针对Android 11之后对访问系统MAC地址的权限进行了修改的问题，导致普通应用在调用NETLINK套接字时会遇到以下几个主要问题：

不允许对NETLINK套接字进行bind操作。
不允许调用RTM_GETLINK功能。
详细说明可以在此链接找到：https://developer.android.com/training/articles/user-data-ids#mac-11-plus

由于上述两个原因，导致在安卓环境下使用Go net包中的net.Interfaces()和net.InterfaceAddrs()时会抛出"route ip+net: netlinkrib: permission denied"错误。
具体 issue 可见：https://github.com/golang/go/issues/40569

为了解决在安卓环境下使用Go net包的问题，我们对其源代码进行了部分改造，以使其能够在Android上正常工作。对于net.InterfaceAddrs()，我们已经完全解决了其中的问题；对于net.Interfaces()，我们只解决了部分问题，目前仍存在以下问题：

只能返回具有IP地址的接口。
不能返回硬件的MAC地址。
但是修复后的net.Interfaces()函数现在与Android API的NetworkInterface.getNetworkInterfaces()保持一致，在大多数情况下可正常使用。

具体修复逻辑包括：

取消了NetlinkRIB()函数中对Netlink套接字的Bind()操作。
根据RTM_GETADDR返回的Index号，使用ioctl获取其网卡的名称、MTU和标志位。