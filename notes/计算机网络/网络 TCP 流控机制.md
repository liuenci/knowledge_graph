#### 转载地址
[TCP流控机制](https://www.cnblogs.com/kubidemanong/p/9987810.html)
#### 为什么需要流量控制
双方在通信的时候，发送方的速率与接收方的速率是不一定相等，如果发送方的发送速率太快，会导致接收方处理不过来，这时候接收方只能把处理不过来的数据存在缓存区里（失序的数据包也会被存放在缓存区里）。

如果缓存区满了发送方还在疯狂着发送数据，接收方只能把收到的数据包丢掉，大量的丢包会极大的浪费网络资源，因此，我们需要控制发送方的发送速率，让接收方与发送方处于一种动态平衡才好。

对发送方发送速率的控制，我们称之为流量控制。

![](https://cdn.nlark.com/yuque/0/2022/jpeg/493161/1647401177924-55d1707c-f915-4e5e-8f3e-c502334dc1d6.jpeg#clientId=u1c0f5e5f-1cdc-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u01aed04d&margin=%5Bobject%20Object%5D&originHeight=610&originWidth=764&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ua50618cc-b464-4496-93d5-b2bd68168b5&title=)

#### 如何控制
接收方每次收到数据包，可以在发送确定报文的时候，同时告诉发送方自己的缓存区还剩余多少是空闲的，我们也把缓存区的剩余大小称之为接收窗口大小，用变量 win 来表示接收窗口的大小。

发送方收到之后，便会调整自己的发送速率，也就是调整自己发送窗口的大小，当发送方接收到接收窗口的大小为 0 时，发送方就会停止发送数据，防止出现大量丢包情况的发生。

![](https://cdn.nlark.com/yuque/0/2022/jpeg/493161/1647401611761-10f42e7e-1495-4a78-bbff-71e45af97bd5.jpeg#clientId=u1c0f5e5f-1cdc-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u655245a6&margin=%5Bobject%20Object%5D&originHeight=573&originWidth=830&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc72c4442-2803-45b5-aa96-38c04d1852c&title=)
#### 发送方何时在继续发送数据
当发送方停止发送数据后，该怎样才能知道自己可以继续发送数据呢？

当接收方处理好数据，接收窗口 win > 0 是，接收方发送通知报文去告诉发送方，告诉他可以继续发送数据了。当发送方收到窗口大于 0 的报文时，就可以继续发送数据。

不过这时候可能会遇到一个问题，假如接收方发送的通知报文，由于某种网络原因，这个报文丢失了，这时候就会引发一个问题：接收方发了通知报文后，继续等待发送方发送数据，而发送方则在等待接收方的通知报文，此时双方会陷入一种僵局。

为了解决这种问题，我们采用了另外一种策略：当发送方收到接收窗口 win = 0 时，这时发送方停止发送报文，并且同时开启一个定时器，每隔一段时间就发送一个测试报文去询问接收方，打听是否可以继续发送数据了，如果可以，接收方就告诉他此时接收窗口的大小；如果接受窗口大小还是为 0 ，则发送方再次刷新启动定时器。

![](https://cdn.nlark.com/yuque/0/2022/jpeg/493161/1647402186840-369f55e4-f38a-4076-94de-91f9d5227d72.jpeg#clientId=u1c0f5e5f-1cdc-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf1b54d2b&margin=%5Bobject%20Object%5D&originHeight=696&originWidth=867&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u8f461677-68b9-4a9e-ab77-bfddbc75dd1&title=)
