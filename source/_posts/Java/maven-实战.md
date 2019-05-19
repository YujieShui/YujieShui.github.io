---
title: maven 实战
toc: true
categories:
  - Java
tags:
  - java
  - maven
abbrlink: cfedc026
date: 2017-12-06 21:11:20
---

Maven 是跨平台的项目管理工具，主要服务于基于Java平台的项目构建、依赖管理和项目信息管理。Maven 的主要思想是**约定优于配置。**通过将约定项目的目录结构，抽象项目的生命周期的方式，将程序员从繁琐的项目构建中解放出来。

<!-- more -->

本文是《Maven 实战》的笔记归纳，找到一本好书就像发现一座宝藏，让人获益匪浅。

*注：本文用的是 maven-3.5.0 版本。*

# Maven 优点和作用

日常工作除了编写源代码，每天有相当一部分时间花在了项目构建中，他们包括项目的编译、运行单元测试、生成文档、打包和部署等烦琐且不起眼的工作。

![构建过程](http://upload-images.jianshu.io/upload_images/2791079-859c04701c2ddf04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**项目自动化构建。**Maven 提供一套规范以及一系列脚本，从清理、编译、测试到生成报告，再到打包和部署实现自动化构建。还提供了插件扩展的方式，进一步简化构建的过程。Maven 还能对项目进行编译、测试、打包，并且将项目生成的构建部署到仓库中。

**Maven 是跨平台的。**对外提供了一致的操作接口，无论在 windows 平台、Linux 平台还是 Mac 上都能用相同的命令进行操作。同时，Maven 项目目录结构、测试用例命名方式等内容都有既定的规则，只要遵循了这些成熟的规则，用户在项目间切换的时候就免去了额外的学习成本。

**Maven 是依赖管理工具。**Java 项目需要依赖许多的 jar 包，随着依赖的增多，版本不一致、版本冲突、依赖臃肿等问题都会接踵而来。Maven 通过`仓库`统一存储这些 jar 包，并通过 `pom 文件`来管理这些依赖。

**Maven 是项目配置工具。**Maven能帮助我们管理原本分散在项目中各个角落的项目信息，包括项目描述、开发者列表、版本控制系统地址、许可证、缺陷管理系统地址等。


# Maven 的仓库

![maven 概念模型](http://upload-images.jianshu.io/upload_images/2791079-d20d3311a6127928.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上是 Maven 的概念模型，前面说过 Maven 能管理众多的 jar 包，并且梳理他们之间的依赖关系。Maven 通过 `pom 文件`和`仓库`进行实现。

## 仓库的作用

如果没有 maven 我们要使用一个 jar 包要从项目的官网寻找下载 jar 到本地，然后再将 jar 包导入到项目中。这样存在几个问题：

1. 去相应的网站寻找 jar 包费精力
2. 下载之后当需要用到某一个 jar 包的时候还要在本地找 jar 包
3. 依赖的 jar 包有多个版本要怎么管理
...

**最好的解决方式就是将这些 jar 包统一管理，每次只要去一个地方找就可以了。**

Maven 就帮我们做了这样一件事情，他提供一个免费的`中央仓库http://repo1.maven.org/maven2`,该中央仓库包含了世界上大部分流行的开源项目。

我们可以从中央仓库下载需要的 jar 包，从`中央仓库`下载的 jar 包会统一保存在 maven 的`本地仓库`中。`本地仓库`在本机的`.m2`文件夹中。

`本地仓库更多相关信息可以去搜索 maven 的安装教程。`

## 更多种类的仓库

![Maven 仓库](http://upload-images.jianshu.io/upload_images/2791079-8a4b71d47412944c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**远程仓库除了中央仓库还有私服和其他公共仓库。**

### 私服

**私服是一种特殊的远程仓库**，它是架设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的Maven用户使用。

![Maven 私服](http://upload-images.jianshu.io/upload_images/2791079-8400377310b692db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图是搭建私服的示意图。私服会将其他公共仓库的 jar 缓存到搭建的服务器上，局域网内的用户还可以将构建的项目直接放到私服上供其他开发人员使用。

架设私服是 Maven 推荐的做法，私服减少中央服务器的压力。如果没有私服，每次请求都需要向中央服务器请求，有了私服之后通过私服的服务器向中央仓库请求，局域网内的用户只需要向私服请求即可。

### 其他公共仓库

比如 Maven 的中央仓库部署在国外，国内访问外网速度不够，我们可以在国内架设 Maven 的公共仓库。

如果仓库 X 可以提供仓库 Y 存储的所有内容，那么就可以认为 X 是 Y 的一个镜像，显然在国内我们需要一个中央仓库的镜像。`http://maven.net.cn/content/groups/public/`是中央仓库，`http://repo1.maven.org/maven2/`在中国的镜像。

Maven 默认是从中央仓库下载文件的，想要让其从其他地方下载文件就要进行配置，这里就需要操作 maven 的 `setting.xml` 文件了。

# setting 文件

在安装好 maven 的基础上，进入 maven 的安装目录，可以看到如下的目录结构：

![Maven 目录](http://upload-images.jianshu.io/upload_images/2791079-e21ac5953d467484.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- bin : mvn 的一些脚本文件
- boot : 含有 plexus-classworlds 类加载器框架
- conf : 配置文件
- lib : maven 所使用的 jar 包（maven 基于 java 开发）

`setting.xml` 文件就在`conf`目录中。**这里的`setting.xml`是 maven 全局的配置文件，不建议修改。**修改之后会影响 maven 的升级等操作。常用的做法是拷贝一份`setting.xml`到 maven 本地仓库的同一目录下，而本地仓库配置在用户目录的`.m2`文件夹中，此时的`setting.xml`就是用户级别的配置文件。

**强烈建议遵循以上规范，避免不必要的麻烦。**

## 自定义本地仓库位置

接下来就来看`setting.xml`的一些配置了。

首先`localRepository`定义本地仓库位置，默认在用户目录下的`.m2/repository`中。

```   
Default: ${user.home}/.m2/repository

<localRepository>
    /path/to/local/repo
</localRepository>
```

## 配置多个远程仓库

前面讲过有中央仓库和其他远程仓库，配置远程仓库就在`repositories`中配置。

```
<repositories>
        <repository>
             <id>jboss</id>
             <name>JBoss Repository</name>
            <url>http://repository.jboss.com/maven2/</url>
            <releases>
                <enabled>true</enabled>
               <updatePolicy>daily</updatePolicy>
            </releases>
            <snapshots>
                 <enabled>false</enabled>
                 <checksumPolicy>warn</checksumPolicy>
           </snapshots>
             <layout>default</layout>
      </repository>
</repositories>
```

在repositories元素下，可以使用repository子 元素声明一个或者多个远程仓库。

## 配置中央仓库镜像

```
＜settings＞ …… 
＜mirrors＞ 
＜mirror＞ 
＜id＞maven.net.cn＜/id＞ 
＜name＞one of the central mirrors in China ＜/name＞
＜url＞ http://maven.net.cn/content/groups/public/ ＜/url＞ 
＜mirrorOf＞central＜/mirrorOf＞ ＜/mirror＞ ＜/mirrors＞ …… 
＜/settings＞
```

## 配置私服作为镜像

```
<mirrors>
     <mirror>
       <id>maven.oschina.net</id>
       <name>maven mirror in China</name>
      <url>http://maven.oschina.net/content/groups/public/</url>
       <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>
```

## 仓库搜索服务

以下网站提供 Maven 仓库搜索功能。

- Sonatype Nexus地址：http://repository.sonatype.org/
- MVNrepository地址：http://mvnrepository.com/

*一般我就用最后一个搜索。*

# Maven 坐标

现在有了仓库统一保管这些 jar 包，剩下的问题就是怎么取了。

不知道你有没有取快递的经验。我们可以这些 jar 包想象成是快递，仓库中保管着这些快递。我们去认领快递需要依靠快递单来确定，一张快递单上会有单号、我们的姓名、手机号等信息。依靠这些信息就不会领错快递了。

这里的快递单就像 Maven 中的 `pom 文件`，单子上的信息就像是 `pom 文件中的坐标系`。

![pom.xml](http://upload-images.jianshu.io/upload_images/2791079-7c2a3316cd5dd610.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Maven 项目规定的项目结构是这样的：

- src/main/java —— 存放项目的.java文件
- src/main/resources —— 存放项目资源文件，如spring, hibernate配置文件
- src/test/java —— 存放所有测试.java文件，如JUnit测试类
- src/test/resources —— 测试资源文件
- target —— 项目输出位置
- pom.xml——maven项目核心配置文件

每个 maven 项目都有 `pom.xml` 文件。Maven坐标为各种构件引入了秩序，任何一个构件都必须明确定义自己的坐标。

![坐标系](http://upload-images.jianshu.io/upload_images/2791079-52d19a7fec46bf6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一组 Maven坐标是通过一些元素定义的，它们是 groupId、artifactId、version、packaging、 classifier：

- groupId：定义当前Maven项目隶属的实际项目，通常为域名反写

- artifactId：该元素定义实际项目中的一个 Maven项目（模块），推荐的做法是使用实际项目名称作为artifactId的前缀。

- version：该元素定义Maven项目当前所处的 版本，

- packaging：该元素定义Maven项目的打包方 式。当不定义packaging的时候，Maven会使用默认值jar。

- classifier：该元素用来帮助定义构建输出的一些附属构件。

通过坐标系我们来保证项目在 Maven 仓库中的唯一性，每次取也不会取错了。

# Maven 依赖

我们自己项目需要用别人的 jar 包，比如 spring。这就是我们的项目依赖于 spring，因此我们通过 pom 来配置这样的依赖关系，这样就能让项目有清晰的结构。

依赖的关系用用`<dependecy>`标签来表示依赖：

![dependencies](http://upload-images.jianshu.io/upload_images/2791079-a7711a8aa8b04871.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*上图说明该项目依赖了 hibernate 等*

## 依赖范围

现在来考虑一种情况，我们在项目开发的过程中用到了 junit 进行测试，也就是说我们的项目依赖于 junit。在项目构建的过程中我们会把 junit 也打包在项目中。但是在生产环境中完全没有必要用到 junit，我们并不想将它发布到生产环境中。

我们可以每次在发布项目之前把他删除了对么？那如果依赖 servlet-api，我们只有在编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器已经提供，也不需要 Maven 重复地引入一遍。

**所以最好是在编译、测试、运行的过程中需要用到什么 jar 包，就让 Maven 去打包什么。**

maven 为此提供了`scope`标签表示`依赖范围`，表示该 jar 包在什么时候需要被使用。

![依赖范围](http://upload-images.jianshu.io/upload_images/2791079-65d7decfb178a767.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- compile：编译依赖范围，使用此依赖范围对于编译、测试、运行三种classpath都有效，即在编译、测试和运行时都要使用该依赖jar包；
- test：测试依赖范围，只对测试有效，表明只在测试的时候需要，在编译和运行时将无法使用该类依赖，如 junit；
- provided：已提供依赖范围。编译和测试有效，运行无效。如servlet-api，在项目运行时，tomcat等容器已经提供，无需Maven重复引入；
- runtime：运行时依赖范围。测试和运行有效，编译无效。如 jdbc 驱动实现，编译时只需接口，测试或运行时才需要具体的 jdbc 驱动实现；
- system：系统依赖范围，使用system范围的依赖时必须通过systemPath元素显示地指定依赖文件的路径，不依赖Maven仓库解析，所以可能会造成建构的不可移植，谨慎使用。

## 依赖传递

依赖范围除了控制classpath，还会对依赖传递产生影响。如果A依赖B，B依赖C，则A对于B是第一直接依赖。B对于C是第二直接依赖。A对于C是传递性依赖。结论是：第一直接依赖的范围和第二直接依赖的范围决定了传递性依赖的范围。

![依赖传递](http://upload-images.jianshu.io/upload_images/2791079-2a80f29f531ca8c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一列是第一直接依赖，第一行是第二直接依赖，中间表示传递性依赖范围。

此外 maven 还提供了`option`和`exclusions`来进一步管理依赖，分别称为`可选依赖`和`排除依赖`。

![可选依赖](http://upload-images.jianshu.io/upload_images/2791079-1bb963176649ce15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*在依赖中添加 <optional> true/false <optional> 表示是否向下传递。*

如上图所示，B 依赖于 X,Y 而 A 依赖于 B，如果 B 不希望将依赖传递给 A 则可以配置 B 中的 X,Y 依赖的`optional`为 true 来阻止依赖的传递。

![排除依赖](http://upload-images.jianshu.io/upload_images/2791079-ebe51ef3d448eb65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再来看一种情况，A 依赖于 B，且 B 将他的依赖 C 传递给了 A。但是 A 依赖了 C 的另一个版本。这个时候 A 可以主动排除 B 给的 C 依赖，转而使用自己需要的版本，这就用到了`exclusions`标签。

用exclusions元素声明排除依赖，exclusions可以包 含一个或者多个exclusion子元素，因此可以排除一个或者多个传递性依赖。

**所以我用主动和被动的方式来区分他们。**

## 依赖冲突

接上面的问题，如果 A 和 B 依赖 C 的不同版本，而且既没有配置`可选依赖`也没有配置`排除依赖`。两个版本都被解析显然是不对的，因为那会造成依赖重复，因此必须选择一个。

**路径最近者优先。**如果直接与间接依赖中包含有同一个坐标不同版本的资源依赖，以直接依赖的版本为准。

**第一声明者优先。**在依赖路径长度相等的前 提下，在POM中依赖声明的顺序决定了谁会被解析使用，顺序最靠前的那个依赖优胜。

上面例子中,A -> C(1.10) 和 A -> B -> C(?),C(1.10)的路径短所以用它。

# Maven 生命周期

Maven的生命周期就是为了对所有的构建过程进行抽象和统一。这个生命周期包含了项目的 清理、初始化、编译、测试、打包、集成测试、 验证、部署和站点生成等几乎所有构建步骤。

初学者往往会以为Maven的生命周期是一个整体，其实不然。Maven拥有三套相互独立的生命周期，它们分别为clean、default和site。clean生命周期的目的是清理项目，default生命周期的目的是构建项目，而site生命周期的目的是建立项目站点。

## clean 生命周期。

clean生命周期的目的是清理项目，它包含三个阶段：

- pre-clean 执行一些需要在clean之前完成的工作 
- clean 移除所有上一次构建生成的文件 
- post-clean 执行一些需要在clean之后立刻完成的工作 

mvn clean 中的clean就是上面的clean，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，mvn clean 等同于 mvn pre-clean clean ，如果我们运行 mvn post-clean ，那么 pre-clean，clean 都会被运行。这是Maven很重要的一个规则，可以大大简化命令行的输入。

## default生命周期

default生命周期定义了真正构建时所需要执 行的所有步骤，它是所有生命周期中最核心的部分，其包含的阶段如下：

- validate 
- generate-sources 
- process-sources 
- generate-resources 
- process-resources 复制并处理资源文件，至目标目录，准备打包。 
- compile 编译项目的源代码。 
- process-classes 
- generate-test-sources 
- process-test-sources 
- generate-test-resources 
- process-test-resources 复制并处理资源文件，至目标测试目录。 
- test-compile 编译测试源代码。 
- process-test-classes 
- test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。 
- prepare-package 
- package 接受编译好的代码，打包成可发布的格式，如 JAR 。 
- pre-integration-test 
- integration-test 
- post-integration-test 
- verify 
- install 将包安装至本地仓库，以让其它项目依赖。 
- deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享。 

运行任何一个阶段的时候，它前面的所有阶段都会被运行，这也就是为什么我们运行mvn install 的时候，代码会被编译，测试，打包。此外，Maven的插件机制是完全依赖Maven的生命周期的，因此理解生命周期至关重要。 

## site生命周期

site生命周期的目的是建立和发布项目站点，Maven能够基于POM所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息。

- pre-site 执行一些需要在生成站点文档之前完成的工作 
site 生成项目的站点文档 
- post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 
- site-deploy 将生成的站点文档部署到特定的服务器上 

这里经常用到的是site阶段和site-deploy阶段，用以生成和发布Maven站点，这可是Maven相当强大的功能，Manager比较喜欢，文档及统计数据自动生成，很好看。 


## 命令与生命周期

mvn clean：该命令调用clean生命周期的clean阶段。实际执行的阶段为clean生命周期的 pre-clean和clean阶段。

mvn test：该命令调用default生命周期的test阶段。实际执行的阶段为default生命周期的 validate、initialize等，直到test的所有阶段。这也解释了为什么在执行测试的时候，项目的代码能够自动得以编译。

mvn clean install：该命令调用clean生命周期 的clean阶段和default生命周期的install阶段。

mvn clean deploy site-deploy：该命令调用 clean生命周期的clean阶段、default生命周期的 deploy阶段，以及site生命周期的site-deploy阶段。

# 聚合与继承

软件设计人员往往会采用各种方式对软件划分模块，以得到更清晰的设计及更高的重用性。当把Maven应用到实际项目中的时候，也需要将**项目分成不同的模块。**

简单的说就是有 A,B 两个模块，现在想要将他们统一管理。**Maven 的聚合特性**能够把项目的各个模块聚合在一起构建，而**Maven的继承特性**则能帮助抽取各模块相同的依赖和插件等配置，在简化POM的同时，还能促进各个模块配置的一致性。

## 聚合

两个子模块希望同时构建。这时，一个简单的需求就会自然而然地显现出来：我们会想要一次构建两个项目，而不是到两个模块的目录下分别执行mvn命令。Maven聚合（或者称为多模块）这一特性就是为该需求服务的。

![项目结构](http://upload-images.jianshu.io/upload_images/2791079-fc655f8309d1609a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图所示`api`是一个模块，`cmd`是一个模块他们都有各自的 pom 文件，其实每一个包都是一个子模块，而最底下的 pom 文件则是统一管理这些子模块。

**他们的配置很简单，我们最好遵循规范。**

**api 的 pom.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>com.shuiyujie.fu</groupId>
        <artifactId>sop</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <artifactId>api</artifactId>
    ...
```
**cmd 的 pom.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>com.shuiyujie.fu</groupId>
        <artifactId>sop</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <artifactId>cmd</artifactId>
```

**聚合 pom.xml**

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.shuiyujie.fu</groupId>
    <artifactId>pom</artifactId>
    <packaging>sop</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>sop</name>
    <url>http://maven.apache.org</url>
    ...    
        <modules>
        <module>base</module>
        <module>core</module>
        <module>server</module>
        <module>persist</module>
        <module>api</module>
        <module>impl</module>
        <module>cmd</module>
    </modules>
    ...
```

观察上面的三个代码清单可以聚合 pom 文件中定义了 `<modules>` 标签，标签中包含的就是各个子模块，并且用子模块的`artifactId`来标记他们。

*注意：聚合 pom 文件的打包方式，即 `packaging` 必须为 pom。*

这样只需要构建聚合 pom 文件即可同时构建在其管理下的多个子模块。

## 继承

消除重复。在面向对象世界中，程序员可以使用类继承在一定程度上消除重复，在Maven的世界 中，也有类似的机制能让我们抽取出重复的配 置，这就是POM的继承。

任然看上面的三个 pom.xml 代码清单，子模块都有一个`parent`标签，这就表明他们继承了一个 pom 文件，而`parent`标签下的其他标签就是一个`坐标系`，通过一个坐标系就能定位一个唯一的项目。

比如上面的子模块继承自`聚合 pom 文件`，所以此时`聚合 pom 文件`也是`父类 pom 文件`。

## 排除父类的依赖

在继承的过程中我们考虑一种情形，我们希望在父类中统一控制 spring 的版本，然后子类继承自父类就可以使用统一版本的 spring 依赖了。但是有些子模块不需要依赖 spring，并不需要从父类继承 spring 的依赖。

**我们可以使用`dependencyManagement` 标签。**

**父类 pom.xml**

```
<dependencyManagement>
        <dependencies>
            <!-- 模块间依赖 start -->
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>core</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>persist</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>api</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>impl</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>server</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>${project.groupId}</groupId>
                <artifactId>cmd</artifactId>
                <version>${project.version}</version>
            </dependency>
            <!-- 模块间依赖 end -->
        </dependencies>
    </dependencyManagement>
```

父类`dependencyManagement`中声明了各个子模块，子模块之间有的会需要相互引用，有的却并不需要。所以在父类中统一配置各个子模块的`groupId`,`artifactId`,`version`等基本信息。

**在`dependencyManagement`中声明的依赖不会在当前pom中引入依赖，也不会再继承他的pom中引入依赖，他的作用只是声明了可能要引入依赖的一些通用信息。**

如果要使用一个子模块要使用其他子模块就可以另外声明，但是不需要指定版本等通用信息，这样就可以减少依赖冲突的发生，代码如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>com.shuiyujie.fu</groupId>
        <artifactId>sop</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <artifactId>api</artifactId>

    <dependencies>
        <dependency>
            <groupId>${project.parent.groupId}</groupId>
            <artifactId>fu-persist</artifactId>
        </dependency>
    </dependencies>
</project>
```

# 总结：Maven 的思想

**Maven 的核心思想是约定优于配置。**

首先，Maven 约定了项目的结构，我们不需要配置 Maven 编译、打包等操作时文件的位置。统一的项目结构降低了学习的成本，让我能将精力集中到了项目本身。

其次，Maven 抽象了项目构建的过程，将其分成一个个生命周期进行管理。通过命令和插件的形式进一步简化操作，又让我们从繁琐的操作解放出来。

# 参考

[《Maven 实战》](https://book.douban.com/subject/5345682/)

[Maven远程仓库的各种配置](http://www.jianshu.com/p/94b060e016a2)

> 本文大部分内容来自于《Maven 实战》一书，想要了解一手信息强烈建议阅读。网上的其他文章基本上都是摘抄《Maven 实战》的部分内容。
> 
> 所以还想说一遍：发现一本好书就像发现了一座宝藏。

