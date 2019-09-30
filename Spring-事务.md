

## Spring事务你真的懂了吗？带你真正理解其底层原理

链接：https://www.bilibili.com/video/av38655504?from=search&seid=15372716880820530612

事务不按照传播机制的预期来执行，有一个可能的主要原因是因为动态代理机制破坏掉了事务传播机制。

如44:50秒左右所说，是因为动态代理机制导致的child的@Transactional注解失效了。
