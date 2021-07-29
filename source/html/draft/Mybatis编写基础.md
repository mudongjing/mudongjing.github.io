> 本文在IDEA下进行操作

首先，这里默认Java环境已配置完毕。由于需要引入各种依赖，这里使用maven。maven配置可自行搜索，但主要的步骤大体一致，下载对应的软件安装后，配置环境变量，另外，为了管理依赖，maven需要设置一个文件夹作为依赖的仓库，未来下载的各种依赖将纳入到该文件夹内。

进入IDEA后，选择maven项目中的quickstart作为空项目。
在一个项目的开发中，需要在中间环节测试当前的开发是否正确，以此需要测试，在新建的项目中，也会包含test目录。
此时，为了配置依赖，其中包括测试类的，日志类的`junit`、`slf4j-api`、`slf4j-log4j12`、`log4j`。
配置的位置在pom.xml文件中的`<dependencies>`包含的范围内，各依赖各自包含在`<dependency>`，具体的格式为maven中的项目，可在[maven](https://mvnrepository.com/ "maven")仓库中查找。
另外，mybatis本身也属于一个依赖，同时添加对应数据库依赖，如mysql。

其中，为了明确编码格式，pom.xml文件加入

```XML
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

mybatis主要是为了方便操作数据库，因此需要明确数据库的连接建立方式。
在**src/main**目录下新键目录**resources**，该目录下，将来会存放各种包含查询语句的xml文件，这些xml是作为对应接口的映射文件。当前需要新键**mybatis-config.xml**，文件内容如下，
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 配置指定使用LOG4J输出日志 -->
	<settings>
		<setting name="logImpl" value="LOG4J"/>
	</settings>
	<!-- 配置包的别名，这样我们在mapper中定义时，就不需要使用类的全限定名称，只需要使用类名即可 -->
	<typeAliases>
		<package name="对应类所在的src/main/java下的目录"/>
	</typeAliases>
	<!-- 数据库配置 -->
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC"/>
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.jdbc.Driver"/>
				<property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
				<property name="username" value="root"/>
				<property name="password" value="root"/>
			</dataSource>
		</environment>
	</environments>

	<!-- mybatis的SQL语句和映射配置文件 -->
	<mappers>
		<package name="resources下的目录"/>
	</mappers>
</configuration>
```
上述内容不需要做过多的解释。
其中"POOLED"指数据库为连接建立连接池，而使用"UNPOOLED"则每次新键连接，在结束后便消除该连接。
如果在IDEA中写xml的文件头时，可能出现url未注册的提醒，快捷键`alt+enter`，选择`fetch external resource`。

由此完成的开发时最基础的准备工作。
此时，需要根据开发需要，在src/main/java下新键对应目录和文件。因为操作数据时，包含数据库中的操作对象格式，以及对应的查询语句。需要将数据库的中对象写作类文件，类内部包含id,商品名或用户名等信息，这些最好作为私有变量，并附带输入或输出的方法，类文件保存在对应的目录下，为实体类。查询语句等用接口替代，接口只需要有名字，对应的具体语句，将写在resources目录下对应的同目录中的同名xml文件中。
需要注意的是，**mybatis-config.xml**文件中的<mappers>标签，保存的就是对应xml文件所在的目录。当然也可以一一将xml文件的路径放在其中。
这里最关键的是，接口与对应的映射xml文件的书写。