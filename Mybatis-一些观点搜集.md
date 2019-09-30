## 1. 如何优雅的使用 Mybatis
来源：http://ityouknow.com/springboot/2016/11/06/spring-boot-mybatis.html
此文中最后有一段作者的看法：

如何选择
两种模式各有特点，注解版适合简单快速的模式，其实像现在流行的这种微服务模式，一个微服务就会对应一个自已的数据库，多表连接查询的需求会大大的降低，会越来越适合这种模式。

老传统模式比适合大型项目，可以灵活的动态生成 Sql ，方便调整 Sql ，也有痛痛快快，洋洋洒洒的写 Sql 的感觉。

## 2. Mapper注解
从mybatis3.4.0开始加入了@Mapper注解，目的就是为了不再写mapper映射文件（那个xml写的是真的蛋疼。。。）

来源：https://blog.csdn.net/phenomenonstell/article/details/79033144
