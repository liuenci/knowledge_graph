[https://zhuanlan.zhihu.com/p/188637127](https://zhuanlan.zhihu.com/p/188637127)
#### Watch Dog 机制
Watch dog 的具体思路是在加锁成功之后，默认加锁 30s，然后每 10s 会去服务器端检查一次锁是否还存在，如果存在就会重新设置过期时间为 30s。
​

#### 官网描述
lockWatchdogTimeout（监控锁的看门狗超时，单位：毫秒）
默认值：30000
监控锁的看门狗超时时间单位为毫秒。该参数只适用于分布式锁的加锁请求中未明确使用 leaseTimeout 参数的情况。如果该看门狗未使用 lockWatchdogTimeout 去重新调整一个分布式锁的 lockWatchdogTimeout 超时，那么这个锁将会变为失效状态。
​

这个参数可以用来避免由 Redisson 客户端节点党纪或者其他原因造成死锁的情况。
​

#### 总结

- 如果需要 Watch Dog 机制生效，加锁时不要设置过期时间。
- Watch Dog 的延时时间可以由 lockWatchdogTimeout  指定默认延时时间，但是不要设置太小。
- Watch Dog 每次延长过期时间是等于 lockWatchdogTime / 3 的时间，每次都是除以 3。
- Watch Dog 通过类似 Netty 的 Future 功能来实现异步延时。
- Watch Dog 最终还是通过 lua 脚本来进行延时。
