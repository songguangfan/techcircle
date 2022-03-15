
实现了无锁版的生产者和消费者框架，它实现了高性能文件拷贝功能。
生产者可以是多个进程，消费者也可以是多个进程。Process类对进程实现了基于面向对象的封装。这个模型是为了解决生产者既要不断产生数据，又要处理数据的一种方案。生产者负责生产数据，消费者负责处理数据。如果生产者产生数据很快，消费者处理数据很慢。那么必然会出现生产者要等待消费者处理完以后，才能继续生产。或者消费者要等待生产者处理完生产数据。无法做到高性能，所以两者间利用了无名管道（pipe）的缓冲，让他们不再直接相互依赖。进程在读取文件时采用pread接口，offset偏移由生产者进程的序号和配置的最大读取长度m_maxlength组成。
其中管道大小是结构体Producer大小的整数倍。这么定义的主要解决了以下问题：
1.如果管道大小不是Producer大小的整数倍，必然会出现生产者生产数据量大的时候写入管道的数据大于管道大小
2.对于消费者，读取的数据大小是固定长度的，那么必然会出现它没有读完，只读取到管道大小数据就直接返回。按照Linux公平调度原则，这个消费者没办法马上读取剩余数据，只能通过read的返回值知道还有多少没有读。
3.管道未满时，生产者进程从阻塞态转为运行态，把剩余数据继续发送到管道返回。其它生产者进程会继续把各自的数据写管道。这样又会带来一个问题。消费者按照固定长度读取，必然会把前一个生产者写入管道的剩余数据和后面生产者写入管道的数据一同读取。从而造成了内容错乱问题。
