# 举个例子

线程A和线程B同时在处理请求，


A的调用链是这样：req1->service->dao->?

B的调用链是这样：req2->service->dao->?

需要注意，dao层的是单例，所以真正实现线程安全的已经是在dao的更底层的模块。

我的猜测是dao的下层会为<b>每次请求生成一个对象</b>，然后在数据库客户端连接池取出一条空闲线程（里面有一个数据库连接对象），
这样子A/B的数据库请求就都分别分配到在不同的连接中，避免多线程同时使用时线程不安全的。
