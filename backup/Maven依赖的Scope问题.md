## 遇到的问题

最近遇到maven使用上的问题。

描述如下：

- 某些依赖，没有从仓库获取，而是本地文件夹内的
- 本机服务正常
- Linux服务器上部署，服务报错，报错内容是ClassNotFound

解决方案：

- 因为这部分代码是从另一个项目复制过来的，包括依赖以及依赖的jar包，放置的位置(resources/lib)

- 对比两个项目，发现plugin部分有个参数不一样

- 修改参数为一致解决问题

  ```xml
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
                  <configuration>
                      <includeSystemScope>true</includeSystemScope>
                  </configuration>
              </plugin>
  ```

## 根源分析

### 核心问题

1. 对Maven定位不清晰
2. 对Maven理论不清楚

### 关于Maven

核心理念：**约定大于配置**；也就是说要有理论基础。

核心定位：项目管理工具，构建是一大主要功能。坐标+依赖是项目的管理，build是构建过程的管理。

### 关于Maven插件

Maven定义了一系列的软件生命周期，clean-validate-compile-test-package-verify-install-site-deploy。

软件生命周期是抽象定义，具体实现是通过插件实现的。类似模板方法。

另外这里有多个约定：

- 插件可以不显式声明，各个Maven版本绑定了各个生命周期阶段的插件
- 一个插件，可能有多个生命周期的执行能力
- 插件内部，有默认绑定的生命周期

还有一点，插件相关的文档不够完善，不好查看和了解，多是遇到问题进行处理。

### 关于Maven依赖范围

这次出问题主要是scope是system，然后package时候，没有包含这些依赖。

首先看下依赖范围是什么：

> 首先需要知道，Maven在编译项目主代码的时候需要使用一套classpath。在上例中，编译项目主代码的时候需要用到spring-core，该文件以依赖的方式被引入到classpath中。其次，Maven在编译和执行测试的时候会使用另外一套classpath。上例中的JUnit就是一个很好的例子，该文件也以依赖的方式引入到测试使用的classpath中，不同的是这里的依赖范围是test。最后，实际运行Maven项目的时候，又会使用一套classpath，上例中的spring-core需要在该classpath中，而JUnit则不需要。
>
> 依赖范围就是用来控制依赖与这三种classpath（编译classpath、测试classpath、运行classpath）的关系。
>
> compile：编译依赖范围。如果没有指定，就会默认使用该依赖范围。使用此依赖范围的Maven依赖，对于编译、测试、运行三种classpath都有效。
>
> system：系统依赖范围。该依赖与三种classpath的关系，和provided依赖范围完全一致。但是，使用system范围的依赖时必须通过systemPath元素显式地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植，因此应该谨慎使用。systemPath元素可以引用环境变量：
>
> ```xml
> ＜dependency＞
> ＜groupId＞javax.sql＜/groupId＞
> ＜artifactId＞jdbc-stdext＜/artifactId＞
> ＜version＞2.0＜/version＞
> ＜scope＞system＜/scope＞
> ＜systemPath＞${java.home}/lib/rt.jar＜/systemPath＞
> ＜/dependency＞
> ```

### 关于spring boot maven plugin

springboot项目，尤其是脚手架项目，都是自带这个插件的，原因在于，springboot项目的方便运行等，通过mvn spring-boot:help可以看到

> This plugin has 7 goals:
>
> spring-boot:build-image
> Package an application into an OCI image using a buildpack.
>
> spring-boot:build-info
> Generate a build-info.properties file based on the content of the current
> MavenProject.
>
> spring-boot:help
> Display help information on spring-boot-maven-plugin.
> Call mvn spring-boot:help -Ddetail=true -Dgoal=<goal-name> to display
> parameter details.
>
> spring-boot:repackage
> Repackage existing JAR and WAR archives so that they can be executed from the
> command line using java -jar. With layout=NONE can also be used simply to
> package a JAR with nested dependencies (and no main class, so not executable).
>
> spring-boot:run
> Run an application in place.
>
> spring-boot:start
> Start a spring application. Contrary to the run goal, this does not block and
> allows other goals to operate on the application. This goal is typically used
> in integration test scenario where the application is started before a test
> suite and stopped after.
>
> spring-boot:stop
> Stop an application that has been started by the 'start' goal. Typically
> invoked once a test suite has completed.

比较核心的是说，使用这个插件打包的jar包，可以直接java -jar启动。

其中package，或者repackage时候，需要includeSystemScope为true，才能引入范围是system的依赖

> ```xml
> <parameter>
> <name>includeSystemScope</name>
> <type>boolean</type>
> <since>1.4.0</since>
> <required>false</required>
> <editable>true</editable>
> <description>Include system scoped dependencies.</description>
> </parameter>
> ```

## 参考

- 《Maven实战》第5章，第7章

- [[spring-boot-maven-plugin插件详解 - 知乎](https://zhuanlan.zhihu.com/p/634098705)](https://zhuanlan.zhihu.com/p/634098705)