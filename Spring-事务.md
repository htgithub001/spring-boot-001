

## Spring事务你真的懂了吗？带你真正理解其底层原理

链接：https://www.bilibili.com/video/av38655504?from=search&seid=15372716880820530612

事务不按照传播机制的预期来执行，有一个可能的主要原因是因为动态代理机制破坏掉了事务传播机制。

如44:50秒左右所说，是因为动态代理机制导致的child的@Transactional注解失效了。

64:00作者说出为什么注解失效的原因，那就是因为只有<b>被代理（代理对象）直接调用后，方法才能被代理</b>。

80:00作者说出解决问题的方案，那就是<b>找到内存中的代理对象，然后用这个对象去调用被代理的方法</b>。

解决方案的代码:
```
void parent() {
  try {
    TestSerivce proxy = (TestSerivce)AopContext.currentProxy()
    proxy.child();
  } catch(Exception e) {
     logger.error("parent 步骤到了child的异常"
  }

...parent自己的业务
}
```
