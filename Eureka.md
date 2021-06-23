# Eureka

## JAX-RS 标准

**JAX-RS: Java API for RESTful Web Services**是一个[Java编程语言](https://zh.wikipedia.org/wiki/Java)的[应用程序界面](https://zh.wikipedia.org/wiki/應用程式介面),支持按照 [表象化状态转变](https://zh.wikipedia.org/wiki/REST) (REST)架构风格创建[Web服务](https://zh.wikipedia.org/wiki/Web服务)[[1\]](https://zh.wikipedia.org/wiki/JAX-RS#cite_note-1). JAX-RS使用了[Java SE 5](https://zh.wikipedia.org/wiki/Java_SE)引入的[Java 标注](https://zh.wikipedia.org/wiki/Java_标注)来简化Web服务客户端和服务端的开发和部署。

**可参考 `javax.ws.rs` 包下的注解**

实现了 jax-rs 标准的框架

* Apache CXF，开源的Web服务框架。
* Jersey， 由Sun提供的JAX-RS的参考实现。
* RESTEasy，JBoss的实现。
* Restlet，由Jerome Louvel和Dave Pawson开发，是最早的REST框架，先于JAX-RS出现。
* Apache Wink，一个Apache软件基金会孵化器中的项目，其服务模块实现JAX-RS规范

> 注意：
>
> SpringMVC 是以 Servlet 为http容器，并自己构建了一套Api，没有遵循 jax-rs 规范

### Jersey 

>  Jersey RESTful WebService框架是一个开源的、产品级别的JAVA框架，支持JAX-RS API并且是一个JAX-RS(JSR 311和 JSR 339)的参考实现

#### @Path

用来为资源类或方法定义URI，当然除了静态URI也支持动态URI

```java
@Path("service") 
public class MyResource {
	@Path("{sub_path}")
    @GET
    public String getResource(@PathParam("sub_path") String resourceName) {
......
```

如果此时客户端请求的URI为http://127.0.0.1:10000/service/sean，则sub_path的值为sean

**@PathParam**用来将请求URI的一部分作为方法参数传入方法中

对URI的动态部分，可以自定义校验正则表达式，如果请求参数校验失败，容器返回404 Not Found。 

```java
@Path("{sub_path:[A-Z]*}")
```

#### @GET

表明被注解的方法响应HTTP GET请求，**@POST**、**@PUT**和**@DELETE**同理

#### **@Consumes**

定义请求的媒体类型，如果不指定，则容器默认可接受任意媒体类型，容器负责确认被调用的方法可接受HTTP请求的媒体类型，否则返回415 Unsupported Media Type

方法级注解将覆盖类级注解

#### **@Produces**

定义响应媒体类型，如果不指定，则容器默认可接受任意媒体类型，容器负责确认被调用的方法可返回HTTP请求可以接受媒体类型，否则返回406 Not Acceptable

方法级注解将覆盖类级注解

#### **@QueryParam**

```java
public String getResource(
		@DefaultValue("Just a test!") @QueryParam("desc") String description) {
	......
}
```

如果请求URI中包含desc参数，例如：http://127.0.0.1:10000/service/sean?desc=123456，则desc参数的值将会赋给方法的参数description，否则方法参数description的值将为**@DefaultValue**注解定义的默认值

#### **@Context**

将信息注入请求或响应相关的类，可注入的类有：Application，UriInfo，Request，HttpHeaders和SecurityContext

#### **@Singleton**和**@PerRequest**

默认情况下，资源类的生命周期是per-request，也就是系统会为每个匹配资源类URI的请求创建一个实例，这样的效率很低，可以对资源类使用**@Singleton**注解，这样在应用范围内，只会创建资源类的一个实例

## Guice 框架

Google 开源的轻量级依赖注入框架

详细教程参考：

[Google Guice 一个轻量级的依赖注入框架](https://www.jianshu.com/p/7fba7b43146a)

[3分钟带你了解轻量级依赖注入框架Google Guice](https://cloud.tencent.com/developer/article/1605665)

[深入剖析Guice](https://developer.aliyun.com/article/518992)

**与 Spirng 的区别**：

| Spring           | Guice                                                        |                                                              |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 使用XML          | 使用将类与类之间的关系隔离到xml中，由容器负责注入被调用的对象，因此叫做依赖注入 | 不使用xml,将类与类之间的关系隔离到Module中，声名何处需要注入，由容器根据Module里的描述，注入被调用的对象。 |
| 使用Annotation   |                                                              | 使用 　　支持自定义Annotation标注，使用Annotation也未必是好事，范型等新特性也未必是好事，目前大多的服务器均不支持 jdk1.5,wls要9以前才支持，而目前的客户由于价格原因也很少选用wls9的，至少我们做过的项目中都没有。功能再强，客户不需要，何用？ |
| 运行效率         | 装载spring配置文件时，需解析xml，效率低，getBean效率也不高，不过使用环境不会涉及到getBean，只有生产环境的时候会用到getBean,在装载spring应用程序的时候，已经完成全部的注射，所以这个低效率的问题不是问题。 | 使用Annotation，cglib, 效率高与spring最明显的一个区别，spring是在装载spring配置文件的时候把该注入的地方都注入完，而Guice呢，则是在使用的时候去注射，运行效率和灵活性高。 |
| 类耦合度         | 耦合度低，强调类非侵入，以外部化的方式处理依赖关系，类里边是很干净的，在配置文件里做文章，对类的依赖性极低。 | 高，代码级的标注，DI标记@inject侵入代码中，耦合到了类层面上来，何止侵入，简直侵略，代码耦合了过多guice的东西，大大背离了依赖注入的初衷，对于代码的可维护性，可读性均不利 |
| 类编写时         | 需要编写xml，配置Bean，配置注入                              | 只需声明为@inject,等着被注入， 　　最后在统一的Module里声明注入方式 |
| 仅支持IOC        | 否，spring目前已经涉猎很多部分                               | 是，目前仅仅是个DI容器                                       |
| 是否易于代码重构 | 统一的xml配置入口，更改容易                                  | 配置工作是在Module里进行，和spring异曲同功                   |
| 支持多种注入方式 | 构造器，setter方法                                           | Field,构造器，setter方法                                     |
| 灵活性           |                                                              | 1,如果同一个接口定义的引用需要注入不同的实现，就要编写不同的Module，烦琐 2,动态注入 如果你想注射的一个实现，你还未知呢，怎么办呢，spring是没办法，事先在配置文件里写死的，而Guice就可以做到，就是说我想注射的这个对象我还不知道注射给谁呢，是在运行时才能得到的的这个接口的实现，所以这就大大提高了依赖注射的灵活性，动态注射。 |
| 与现有框架集成度 | 1， 高，众多现有优秀的框架（如struts1.x等）均提供了spring的集成入口，而且spring已经不仅仅是依赖注入，包括众多方面。 　　2， Spring也提供了对Hibernate等的集成，可大大简化开发难度。 　　3， 提供对于orm,rmi,webservice等等接口众多，体系庞大 | 1，可以与现有框架集成，不过仅仅依靠一个效率稍高的DI，就想取代spring的地位，有点难度。 |
| 配置复杂度       | 在xml中定位类与类之间的关系,难度低                           | 代码级定位类与类之间的关系,难度稍高                          |