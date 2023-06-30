In response to the modifications made to the permissions for accessing system MAC addresses in Android 11, ordinary applications encounter several main issues when using NETLINK sockets:

Not allowing bind operations on NETLINK sockets.
Not permitting the use of the RTM_GETLINK functionality.
For detailed information, please refer to: https://developer.android.com/training/articles/user-data-ids#mac-11-plus

As a result of the aforementioned reasons, using net.Interfaces() and net.InterfaceAddrs() from the Go net package in the Android environment leads to the "route ip+net: netlinkrib: permission denied" error. You can find specific issue details here: https://github.com/golang/go/issues/40569

To address the issue of using the Go net package in the Android environment, we have made partial modifications to its source code to ensure proper functionality on Android. We have fully resolved the issues with net.InterfaceAddrs(). However, for net.Interfaces(), we have only addressed some problems, as the following issues still remain:

It can only return interfaces with IP addresses.
It cannot return hardware MAC addresses.
Nevertheless, the fixed net.Interfaces() function now aligns with the Android API's NetworkInterface.getNetworkInterfaces() and can be used normally in most scenarios.

The specific fix logic includes:

Removing the Bind() operation on Netlink sockets in the NetlinkRIB() function.
Using ioctl based on the Index number returned by RTM_GETADDR to retrieve the network card's name, MTU, and flags.