---
title: 发布java项目到maven中央库
layout: post
---

## What is Maven?
对我来说，Maven就是一套项目管理框架:
	一个项目对象模型 (Project Object Model)，
	一组标准集合，
	一个项目生命周期(Project Lifecycle)，
	一个依赖管理系统(Dependency Management System)，
	和用来运行定义在生命周期阶段(phase)中插件(plugin)目标(goal)的逻辑。

## Maven 中央库
Maven可以自己搭建，也可以使用线上搭建好的Maven平台，Maven 中央库就是一个在线提供java项目托管的Maven平台。将项目发布到Maven中央库后，其他人就可以根据项目的名称或Id直接获取到相应的jar包，同时Maven还解决了项目依赖的问题。（类似Python社区的pypi）

## 如何将jar包发布到Maven中央库

### 注册Maven中央库账号
在<http://oss.sonatype.org>注册账号（注：Sonatype是Maven中央库的管理系统），注册之后获得用户名，密码。

### 向Maven中央库申请一个仓库
在Sonatype的JIRA系统中，创建一个issue（选择Project: Community Support - Open Source Project Repository Hosting; Issue Type: New Project)，告诉Sonatype管理员，你想托管一个项目到Sonatype上。注意该Issue创建完之后，只有管理员有更改权限，因此，小心不要写错信息(详见[官方说明](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide))。创建完之后，需要等待Sonatype管理员审核，一般不超过2个工作日，一旦审核通过，会在该Issue上标明Resolved，这就是说中央库已经准备好，可以随时上传自己的文件了。 

### 本地Maven配置[官方说明](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide)
本地需要安装工具：JDK && Maven，Maven安装方法详见[文档](http://coaku.qiniudn.com/maven_guide.pdf)，以及GPG签名工具

告诉Maven我们之前在Maven中央库上申请的帐号，即在maven工具的配置文件~/.m2/settings.xml中添加如下：

    <server>
        <id>sonatype-nexus-snapshots</id>
        <username>USERNAME</username>
        <password>PASSWORD</password>
    </server>
    <server>
        <id>sonatype-nexus-staging</id>
        <username>USERNAME</username>
        <password>PASSWORD</password>
    </server>

然后就是修改位于项目根目录的pom.xml文件，这个文件是Maven行为的总指挥配置文件，类似于Makefile，用来告诉Maven关于项目编译，打包，签名，依赖对象在Maven中央库上的位置等重要信息。

pom.xml文件中一般需要以下配置：

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
        <groupId>com.coaku</groupId>
        <artifactId>sdk</artifactId>
        <version>1.1.1-SNAPSHOT</version>
        <packaging>jar</packaging>
        <name>java-sdk</name>
        <url>http://www.coaku.com/</url>
        <description> Qiniu Resource (Cloud) Storage SDK demo for Java</description>

        <licenses>
            <license>
                <name>The MIT License</name>
                <url>http://opensource.org/licenses/MIT</url>
            </license>
        </licenses>
        
        <parent>
            <groupId>org.sonatype.oss</groupId>
            <artifactId>oss-parent</artifactId>
            <version>7</version>
        </parent>
        
        <scm>
            <connection>scm:git:git@github.com:coaku/java-sdk.git</connection>
            <developerConnection>scm:git:git@github.com:coaku/java-sdk.git</developerConnection>
            <url>git@github.com:coaku/java-sdk.git</url>
        </scm>
        
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.1</version>
                    <configuration>
                        <source>1.6</source>
                        <target>1.6</target>
                        <encoding>utf-8</encoding>
                    </configuration>
                </plugin>
            
            <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-gpg-plugin</artifactId>
                    <executions>
                            <execution>
                                    <id>sign-artifacts</id>
                                    <phase>verify</phase>
                                    <goals>
                                        <goal>sign</goal>
                                    </goals>
                            </execution>
                    </executions>
            </plugin>
            
            </plugins>
	    </build>

        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        </properties>

        <denpendencies>
                ...
        </denpendencies>
    
    </project>


### 使用GPG对java项目包进行签名[官方说明](https://docs.sonatype.org/display/Repository/How+To+Generate+PGP+Signatures+With+Maven)

    > gpg --gen-key                 // 填写一系列信息后生成一对公密钥，这期间需要手动输入一个passphrase密码，后面签名时会用到

    > gpg --list-keys               // 列出生成的key信息
    /home/coaku/.gnupg/pubring.gpg
    ------------------------------
    pub   2048R/11FCA5B2 2013-07-10
    uid                  Qiniu Resource Storage <qbox.team@gmail.com>
    sub   2048R/DAE43050 2013-07-10

    > gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 11FCA5B2    // 将公钥上传到一台公钥服务器供别人下载

    其他人可以通过如下方式获取公钥：
    > gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 11FCA5B2

### 使用mvn(Maven)进行打包上传
maven上的发布分为两种发布版本，一种是SNAPSHOT版本，一种是正式RELEASE版本，这两种发布过程不一样，但都要求pom.xml配置中项目的版本号后面要带有"-SNAPSHOT"格式，只是在正式RELEASE时候，maven会自动将"-SNAPSHOT"后缀去掉。

上传SNAPSHOT版本：

    > mvn clean deploy -Dgpg.passphrase=<PASSPHRASE>      // <passphrase>即之前生成gpg公密钥时候手动输入的密码，下面也是

上传正式RELEASE版本：

    > mvn release:clean
    > mvn release:prepare -Darguments=-Dgpg.passphrase=<PASSPHRASE>
    > mvn release:perform -Darguments=-Dgpg.passphrase=<PASSPHRASE>

### 正式 Release 项目
在Sonatype的管理员将上传的项目正式同步到Maven中央库之前，项目会存在于一个中间状态的仓库中(Staging Repository)，起始状态为open，我们需要将它变为close。步骤[如此](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8a.ReleaseIt).

在状态变为close的过程中，Maven会对项目进行一系列的检查，如果通过了检查，状态会顺利切换到close。

切换到close后，就能够正式release这个项目，切换方法同close一样，点击release按钮即可。

### 如果是第一次发布，需要通知管理员激活该仓库和Maven中央库的同步
回到之前发布的那个issue上，回复给管理员，说自己的仓库已经release了，管理员在收到回复后会激活这个仓库和Maven中央库的同步工作。


## 注意的问题

1. pom.xml中项目版本号一定要是"x.x.x-SNAPSHOT"格式
2. pom.xml中对scm(软件配置管理)的配置要正确，如果是github，那么需要以ssh认证的方式配置对github的访问，[详情](https://help.github.com/articles/generating-ssh-keys)，因为mvn在上传过程中涉及到对github的push动作。
3. 在使用mvn进行上传之前，要确保本地的代码和scm中的最新版本的一致。
