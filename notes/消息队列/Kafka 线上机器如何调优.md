#### 生产者配置


```
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092"); 
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("buffer.memory", 67108864); 
props.put("batch.size", 131072); 
props.put("linger.ms", 100); 
props.put("max.request.size", 10485760); 
props.put("acks", "1"); 
props.put("retries", 10); 
props.put("retry.backoff.ms", 500);

KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
```


#### 内存缓冲的大小 buffer.memory Kafka Manager


kafka的客户端发送数据到服务器，一般都是要经过缓冲的，也就是说，你通过kafkaProducer发送出去的消息都是先进入到客户端本地的内存缓冲里，然后把很多消息收集成一个一个的batch，再发送到broker上去的。所以这个 buffer.memory 的本质就是用来约束 kafkaProducer 能够使用的内存缓冲的大小的，他的默认值是 32mb.


生产环境上如何设置内存缓冲的大小？


首先要明确的一个点就是，内存缓冲里大量的消息回缓冲在里面，形成一个一个的 batch,每个 batch 里包含多条消息。然后 KafkaProducer 有一个 Sender 线程会把多个 Batch 打包成一个 Request 发送到 Kafka 服务器上去。


如果内存设置的太小：会导致消息快速的写入内存缓冲里面，但是 Sender 线程来不及把 Request 发送到 Kafka 服务器上。这样会造成内存缓冲很快就会被写满？一旦被写满，就会阻塞用户线程，不让继续往 Kafka 写消息了。


所以生产环境需要自己测算一下，假设内存缓冲就32mb，每秒写 300 条消息到内存缓冲，是否会经常把内存缓冲写满？经过这样的压测，就能调整出一个合理的内存大小。


开发的调优方式：写一个接口，然后里面通过线程池启动N个线程，并发同时写入消息，然后观察每次写入消息的成功时间间隔，如果间隔变大，也就是花费的时间太长了，说明被写满了，线程被堵塞了。这个和每个系统的并发量有关，而且还和每个消息的大小有关。


#### 多少数据打包成一个batch合适：batch.size


batch.size 决定的是你的每个 batch 要存放多少数据就可以发送出去了。比如说你要是设置 batch 为 16kb 的大小，那么里面凑够了 16kb 的数据就可以发送了。这个参数的默认值是 16kb,一般可以把这个参数调节大一些，然后利用自己的生产环境发消息的负载来测试一下，比如发送消息的频率就是每秒300条，那么如果比如 batch.size 调节到了 32KB，或者 64KB，是否可以提升发送消息的整体吞吐量。


因为理论上来说，提升 batch 的大小，可以允许更多的数据缓冲在里面，那么一次 Request 发送出去的数据量就更多了，这样吞吐量可能会有所提升。但是这个东西也不能无限的大，过于大了之后，要是数据老是缓冲在 Batch 里面迟迟不发送出去，发送消息的延迟就会很高，比如说一条消息进入了 Batch，但是要等待 5s batch 才凑满了，那这条消息的延迟就是 5s.所以这里需要按照生产环境的发消息的速率，调节不同的 batch 大小自己测试一下最终出去的吞吐量以及消息的延迟，设置一个最合理的参数。


#### 如果一个 batch 无法凑满怎么办：linger.ms 关键参数


如果一个 batch 无法凑满，可以引入另外一个参数，"linger.ms",他的含义就是当一个 batch 被创建之后，最多过多久，不管这个 batch 有没有写满，都必须发送出去了。


假设一个 batch.size 是 16kb,但是现在某个低峰时间段，发送消息很慢。这就导致了 batch 被创建之后，陆陆续续有消息进来，但是迟迟无法凑够 16kb，难道此时就一直等着吗？


当然不是，假设你现在设置 linger.ms 是 50ms,那么只要这个 batch 从创建开始到现在已经过了 50ms了，哪怕他还没满 16kb,也要发送他出去了。所以 linger.ms 决定了你的消息一旦写入一个 batch ,最多等待这么多时间，他一定会跟着这个 batch 一起发送出去。


这样可以避免一个 batch 迟迟凑不满，导致消息一直挤压在内存里发送不出去的情况。


这个参数需要配合 batch.size 一起来设置。举个例子，首先假设你的 batch 是 32kb,如果 20ms 就会凑够一个 batch。那么你的 linger.ms 可以设置为 25ms，也就是说正常来说，大部分的 batch 在 20ms 内都会凑满，但是你的 linger.ms 可以保证，哪怕入到低峰时期，20ms 凑不满一个 batch,还是会在 25ms 之后强制 batch 发送出去。


如果要是你把 linger.ms 设置的太小了，比如说默认就是 0ms,或者你设置个 5ms,那可能导致你的 batch 虽然设置了 32kb，但是经常是还没有凑够 32kb 的数据， 5ms 之后就直接强制 batch 发送出去，这样也不好，会导致你的 batch 形同虚设，一直凑不满数据。


#### 最大请求大小 max.request.size


这个参数决定了每次发送给kakfa服务器请求的最大大小，同时也会限制你一条消息的最大大小也不能超过这个参数设置的值，这个可以根据自己的消息的大小来灵活的调整。


举个例子：公司发送的消息都是那种比较大的报文消息，每条消息都是很多的数据，一条消息可能就要 20kb，此时你的batch.size是不是就需要调节大一些？比如设置一个512kb,然后你的buffer.memory也可以给大一点，设置成128m,只有这样，才能让你在大消息的场景下，还能使用 batch 打包多条消息的机制。


#### 重试机制 retries 和 retries.backoff.ms


这两个是 kafka 的重试机制，也就是说一个请求失败了可以重试几次，每次重试的间隔是多少毫秒。


各个业务可以按需设置。


#### 确认机制 acks


这个配置是当一次purduce请求被认为完成是的确认值。特别是，多少个其他brokers必须已经提交了数据到他们的log并且向他们的leader确认了这些信息。典型的值包括


0:表示purducer从来不等待来自broker的确认信息。这个选择提供了最小的时延但同时风险最大，因为当服务器宕机是，数据将会丢失。


1:表示leader replica已经接收到了数据的确认信息。这个选择时延较小同时确保了server确认接收成功


-1:purducer会获得所有同步replica都接收到数据的确认。同时时延最大，然后这种方式并没有完全消除丢失消息的风险，因为同步replicas的数量可能是1。
