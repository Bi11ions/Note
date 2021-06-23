# Maven

## mvn 指令

### 1. 格式

`mvn [plugin-name]:[goal-name]`

### 2. 参数

| 参数 | 说明                                                     |
| ---- | -------------------------------------------------------- |
| -D   | 指定参数，如 -Dmaven.test.skip=true 跳过单元测试         |
| -P   | 指定 Profile 配置，可以用于区分环境                      |
| -e   | 显示maven运行出错的信息                                  |
| -o   | 离线执行命令,即不去远程仓库更新包                        |
| -X   | 显示maven允许的debug信息                                 |
| -U   | 强制去远程更新snapshot的插件或依赖，默认每天只更新一次。 |

### 3. 常用命令

| 命令                     | 参数                                                         | 说明                                   |
| ------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| mvn archetype:create     |                                                              | 创建maven项目                          |
|                          | -DgroupId=packageName                                        | 指定 group                             |
|                          | -DartifactId=projectName                                     | 指定 artifact                          |
|                          | -DarchetypeArtifactId=maven-archetype-webapp                 | 创建web项目                            |
| mvn archetype:generate   |                                                              | 创建maven项目                          |
| mvn validate             |                                                              | 验证项目是否正确                       |
| mvn package              |                                                              | maven 打包                             |
| mvn jar:jar              |                                                              | 只打jar包                              |
| mvn source:jar           |                                                              | 生成源码jar包                          |
| mvn generate-sources     |                                                              | 产生应用需要的任何额外的源代码         |
| mvn compile              |                                                              | 编译源代码                             |
| mvn test-compile         |                                                              | 编译测试代码                           |
| mvn test                 |                                                              | 运行测试                               |
| mvn verify               |                                                              | 运行检查                               |
| mvn clean                |                                                              | 清理maven项目                          |
| mvn eclipse:eclipse      |                                                              | 生成eclipse项目                        |
| mvn eclipse:clean        |                                                              | 清理eclipse配置                        |
| mvn idea:idea            |                                                              | 生成idea项目                           |
| mvn install              |                                                              | 安装项目到本地仓库                     |
| mvn:deploy               |                                                              | 发布项目到远程仓库                     |
| mvn integration-test     |                                                              | 在集成测试可以运行的环境中处理和发布包 |
| mvn dependency:tree      |                                                              | 显示maven依赖树                        |
| mvn dependency:list      |                                                              | 显示maven依赖列表                      |
| mvn dependency:sources   |                                                              | 下载依赖包的源码                       |
| mvn install:install-file | -DgroupId=packageName -DartifactId=projectName -Dversion=version -Dpackaging=jar -Dfile=path | 安装本地jar到本地仓库                  |

### 4. web 项目相关命令

| 命令                             | 说明              |
| -------------------------------- | ----------------- |
| mvn tomcat:run                   | 启动tomcat        |
| mvn jetty:run                    | 启动jetty         |
| mvn tomcat:deploy                | 运行打包部署      |
| mvn tomcat:undeploy              | 撤销部署          |
| mvn tomcat:start                 | 启动web应用       |
| mvn tomcat:stop                  | 停止web应用       |
| mvn tomcat:redeploy              | 重新部署          |
| mvn war:exploded tomcat:exploded | 部署展开的war文件 |

## scope 的作用

### 一、说明

表明依赖的作用范围

使用示例：

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>3.2.1.RELEASE</version>
            <scope>test</scope>
        </dependency>
```

### 二、取值

| scope取值                 | 有效范围（compile, runtime, test） | 依赖传递 | 例子        |
| :------------------------ | :--------------------------------- | :------- | :---------- |
| **compile**（**默认值**） | all                                | 是       | spring-core |
| **provided**              | compile, test                      | 否       | servlet-api |
| **runtime**               | runtime, test                      | 是       | JDBC驱动    |
| **test**                  | test                               | 否       | JUnit       |
| **system**                | compile, test                      | 是       |             |

> **compile** ：**为默认的依赖有效范围**。如果在定义依赖关系的时候，没有明确指定依赖有效范围的话，则默认采用该依赖有效范围。此种依赖，在编译、运行、测试时均有效。
>
> **provided** ：**在编译、测试时有效，但是在运行时无效**。例如：servlet-api，运行项目时，容器已经提供，就不需要Maven重复地引入一遍了。
>
> **runtime** ：**在运行、测试时有效，但是在编译代码时无效。**例如：JDBC驱动实现，项目代码编译只需要JDK提供的JDBC接口，只有在测试或运行项目时才需要实现上述接口的具体JDBC驱动。
>
> **test** ：**只在测试时有效**，例如：JUnit。
>
> **system** ：**在编译、测试时有效，但是在运行时无效。**和provided的区别是，使用system范围的依赖时必须通过systemPath元素显式地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植，因此应该谨慎使用。systemPath元素可以引用环境变量

### 三、scope的依赖传递

A–>B–>C。当前项目为A，A依赖于B，B依赖于C。知道B在A项目中的scope，那么怎么知道C在A中的scope呢？

答案是： 

当 C 是 test 或者 provided 时，C 直接被丢弃，A 不依赖 C； 
否则 A 依赖 C，C 的 scope 继承于 B 的 scope。