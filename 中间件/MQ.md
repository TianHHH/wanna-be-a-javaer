# RabbitMQ

## RabbitMQ中怎么实现延迟队列

**<font style="color:blue">第一种是通过 "TTL + 死信队列" 的方式实现延迟消息</font>**. 实现思路就是:

- 先创建一个用于延迟的普通队列, 并配置消息的 TTL(也就是过期时间). 需要注意的是这个队列是没有任何绑定的消费者的, 这样消息因为只能在队列中等待 TTL 到期, 所以自然就会过期
- 当消息过期后, RabbitMQ会把消息变为"死信", 投递到该队列绑定的死信交换机中
- 死信交换机会将消息转发到死信队列中, 然后就可以通过订阅该死信队列消费消息, 从而实现延迟队列

这种方式的缺点就是不能实现随机延迟任务, 因为在一个队列上只能设置统一的一个过期时间, 所以只能实现固定过期时间的延迟任务



**<font style="color:blue">第二种方式就是使用 RabbitMQ 提供的官方延迟插件</font>**

​	这种方式就是先在 Github 上根据自己的 RabbitMQ 版本下载相应版本的插件

​	然后在**将插件放到 RabbitMQ 服务器的一个叫 plugins 插件目录中, 再通过`rabbitmq-plugins enable <插件>`的方式启用插件**

​	最后通过`systemctl`命令重启RabbitMQ服务之后, 插件就生效了

- 启用延迟插件后可以声明一种特殊类型的交换机 -- `x-delayed-message`
- **并且在其中<font style="color:blue">每条消息都可以单独设置 `x-delay` 这个header, 指定要延迟的毫秒数</font>**. 系统会自动延迟相应时间后再投递到目标队列

> 这种方式其实也有缺点, 可以参考下面的内容
>
> https://www.nowcoder.com/discuss/642795083435081728?sourceSSR=search















