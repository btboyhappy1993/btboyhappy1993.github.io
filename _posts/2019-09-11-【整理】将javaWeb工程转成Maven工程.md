---
layout: post
title:  "【整理】将javaWeb工程转成Maven工程"
date:   2019-09-11 23:31
categories: 整理
permalink: /archivers/20190911-4
---

之前碰到过很多javaWeb工程，直接使用lib文件夹管理jar包，缺点多多，jar包版本混乱，不堪其扰，故创建pom、整理成maven管理。

而且这种工程打包基本都是使用eclipse等开发工具打包，不方便使用自动构建工具，使用maven管理可以使用它的打包工具。

 1. 创建pom文件，将配置信息和依赖jar包信息写入其中
 jar包相关信息可以去[该网址](http://mvnrepository.com/)去查找。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.chongtong</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>demo</description>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <jdk.version>1.8</jdk.version>
        <spring.version>3.0.5.RELEASE</spring.version>
    </properties>
    <dependencies>
        <!-- core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- web -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-servlet</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>demo</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.2</version>
                <configuration>
                    <source>${jdk.version}</source>
                    <target>${jdk.version}</target>
                    <encoding>utf8</encoding>
                </configuration>
            </plugin>
            <!-- The configuration of maven-assembly-plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.1.0</version>
                <!-- The configuration of the plugin -->
                <configuration>
                    <!-- Specifies the configuration file of the assembly plugin -->
                    <descriptors>
                        <descriptor>src/main/assembly/package.xml</descriptor>
                    </descriptors>
                    <!-- 设为 FALSE, 防止 WAR 包名加入 assembly.xml 中的 ID -->
                    <appendAssemblyId>false</appendAssemblyId>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <warSourceDirectory>${basedir}/src/main/webapp
                    </warSourceDirectory>
                    <webXml>${basedir}/src/main/webapp/WEB-INF/web.xml</webXml>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>findbugs-maven-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <findbugsXmlOutput>true</findbugsXmlOutput>
                    <xmlOutput>true</xmlOutput>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <packaging>war</packaging>
</project>
```

 2. 创建打包文件`package.xml`，将打包策略写入其中

```xml
 <assembly>
    <id>${project.artifactId}-assembly-${project.version}</id>
    <!-- 默认为 TRUE, 设为 FALSE, 防止将 ${project.finalName} 作为根目录打进 WAR 包 -->
    <!-- TRUE 结构: ${project.finalName}.war/${project.finalName}/WEB-INF -->
    <!-- FALSE 结构: ${project.finalName}.war/WEB-INF -->
    <includeBaseDirectory>false</includeBaseDirectory>
    <!-- 设置为 WAR 包格式 -->
    <formats>
        <format>war</format>
    </formats>
    <fileSets>
        <!-- 将 target/classes 下的文件输出到 WEB-INF/classes -->
        <fileSet>
            <directory>${project.build.outputDirectory}</directory>
            <outputDirectory>WEB-INF/classes</outputDirectory>
        </fileSet>
        <!-- 将 webapp 下的文件输出到 WAR 包 -->
        <fileSet>
            <directory>${project.basedir}/src/main/webapp</directory>
            <outputDirectory>/</outputDirectory>
        </fileSet>
    </fileSets>
    <!-- 将项目依赖的JAR包输出到 WEB-INF/lib -->
    <dependencySets>
        <dependencySet>
            <outputDirectory>WEB-INF/lib</outputDirectory>
            <excludes>
                <exclude>${project.groupId}:${project.artifactId}:war</exclude>
            </excludes>
        </dependencySet>
    </dependencySets>
</assembly> 
```
