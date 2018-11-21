### Maven入门

`Maven`是一个项目管理工具，`Maven`是基于项目对象模型（`POM`）。

要使用`Maven`首先要做的事就是环境变量配置。

#### Maven目录结构

`Maven`定义了一个标准的目录结构。

```
- src
  - main
    - java
    - resources
    - webapp
  - test
    - java
    - resources
- pom.xml
- target
```

`src `目录是源代码和测试代码的根目录。

`main `目录是与源代码相关的根目录到应用程序本身，而不是测试代码。

`test `目录包含测试源代码。

`main`和`test`下的` java `目录包含`Java`代码的应用程序本身是在`main`和用于测试的`Java`代码。

`resources `目录包含您项目所需的资源。

`target `目录由`Maven`创建。它包含所有编译的类，`JAR`文件等。

当执行` mvn clean `命令时，`Maven`将清除目标目录。

`webapp `目录包含`Java Web`应用程序，如果项目是`Web`应用程序。

`webapp `目录是`Web`应用程序的根目录。`webapp`目录包含` WEB-INF `目录。

#### pom.xml文件

`Project Object Model`项目对象模型，`Maven`的核心配置文件`pom.xml`，与构建过程相关的一切设置都在这个文件中进行配置。

这里是一个最小的`POM`文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.flwcy.demo</groupId>
    <artifactId>demo-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo-api</name>
</project>
```

所有POM文件都需要项目XML元素和三个必填字段:` groupId `，` artifactId `，`version`。

`modelVersion `元素设置`POM`模型的版本。它必须匹配您使用的Maven版本。版本4.0.0匹配`Maven`版本2和3。

+ `groupId`组织名，一般是公司网址的反写+项目名
+ `artifactId`项目名
+ `version`项目当前版本号

#### Maven依赖

当项目越来越大，我们将需要越来越多的外部依赖。`Maven`将下载它们并将它们放在您的本地`Maven`存储库中。如果我们想要在工程中引入某个`jar`包，只需要在`pom.xml` 中引入其`jar`包的坐标即可。比如引入`junit`的依赖：

```xml
<project ...>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.flwcy.demo</groupId>
    <artifactId>demo-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

#### Maven常用命令

1. `mvn compile`：编译项目
2. `mvn test`：测试命令,或执行`src/test/java/`下`junit`的测试用例
3. `mvn package`：项目打包工具,会在模块下的`target`目录生成`jar`或`war`等文件
4. `mvn clean`：清理项目生产的临时文件,一般是模块下的`target`目录
5. `mvn install`：模块安装命令，将打包的的`jar/war`文件复制到你的本地仓库中,供其他模块使用`-Dmaven.test.skip=true`跳过测试(同时会跳过`test compile`)

#### Maven坐标

`Maven`坐标其实是一组用来标识构件唯一性的元素组合，通过坐标，我们就能从`Maven`库从引用指定的`jar`包、`war`包等`Maven`构件。

在实际的生活中，我们可以将地址看成是一种坐标。就像快递小哥根据你填写的地址和电话将快递送给你。

`Maven坐标`是通过`groupId`、`artifactId`、`version`、`packaging`、`classfier`这些元素来定义的。我们在平时的开发中一般只需要使用必要的几个元素就好了。

+ **groupId** ：定义当前`Maven`项目隶属的实际项目。
+ **artifactId** : 该元素定义当前实际项目中的一个`Maven`项目（模块）。
+ **version** : 该元素定义`Maven`项目当前的版本。
+ **packaging** ：定义`Maven`项目打包的方式。
+ **classifier**: 该元素用来帮助定义构建输出的一些附件。

上述5个元素中，`groupId`、`artifactId`、`version`是必须定义的，`packaging`是可选的（默认为jar），而`classfier`是不能直接定义的，需要结合插件使用。

#### 仓库

在`Maven`中，任何一个依赖、插件或者项目构建的输出，都可以称之为构件。`Maven`在某个统一的位置存储所有项目的共享的构件，这个统一的位置，我们就称之为仓库。（仓库就是存放依赖和插件的地方）。任何的构件都有唯一的坐标，`Maven`根据这个坐标定义了构件在仓库中的唯一存储路径。

`Maven`的仓库只有两大类：

1. 本地仓库
2. 远程仓库

##### 本地仓库

`Maven`默认的本地仓库路径为`${user.home}/.m2/repository`

> 注：`Maven`的本地仓库，在安装`Maven`后并不会创建，它是在第一次执行`Maven`命令的时候才被创建

如何更改`Maven`默认的本地仓库的位置：这里要引入一个新的元素**localRepository，它是存在于maven的settings.xml文件中**。

修改`settings.xml`文件，配置本地仓库路径，在`settings.xml`中新增一行如下内容：

```xml
  <localRepository>E:\.m2\repository</localRepository>
```

并将`settings.xml`拷贝到`E:\.m2`下（可以根据自己的情况选择合适的路径）。

##### 远程仓库

由于最原始的本地仓库是空的，`Maven`必须知道至少一个可用的远程仓库，才能在执行`Maven`命令的时候下载到需要的构件。中央仓库就是这样一个默认的远程仓库，`Maven`的安装文件自带了中央仓库的配置。

在平时的开发中，我们往往不会使用默认的中央仓库，默认的中央仓库访问的速度比较慢，访问的人或许很多，有时候也无法满足我们项目的需求，因此通常我们会配置一个国内镜像仓库。

使用方法：在`setting.xml`文件的`mirrors`节点下添加即可。

```xml
<mirror>  
  <id>alimaven</id>  
  <name>aliyun maven</name>  
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
  <mirrorOf>central</mirrorOf>          
</mirror> 
```

