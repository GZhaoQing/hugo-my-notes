---
title: "Maven使用经验总结"
date: 2021-11-17T11:17:34+08:00
draft: false
tags: [Maven]
categories: [Maven]
---

### 配置

Maven的conf/settings.xml文件中可以配置本地仓库、中央仓库、使用的默认jdk版本

关键配置如下

```xml
<localRepository>E:/mvnRepo</localRepository>

<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>   
</mirror>

<profile>    
    <id>jdk-1.8</id>    
    <activation>    
        <activeByDefault>true</activeByDefault>    
        <jdk>1.8</jdk>    
    </activation>    
    <properties>    
        <maven.compiler.source>1.8</maven.compiler.source>    
        <maven.compiler.target>1.8</maven.compiler.target>    
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>   
    </properties>    
</profile>

```

### Eclipse插件

对于新的eclipse工作空间，记得在全局配置中设置本地的Maven。



IDEA中maven插件会把命令都列出来，eclipse中除了默认命令，都在Run as->Maven build命令中自己写，插件会保存每个项目执行过的命令。

Name一栏是自定义的名称。

Goals一栏就是写命令的地方，填写的是去掉mvn之后的部分。在

命令行中如果是这样：mvn package，则在Goals中填package就可以了。

如果用到了插件：mvn spring-boot:repackage，则Goals中填spring-boot:repackage

插件的语法是mvn [plugin-name]:[goal-name]





### 有用的插件

#### maven-war-plugin

此例中工程自己（非Maven）的三方libs也打入war中

```xml
			<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <configuration>
                    <webResources>
                        <resource>
                            <directory>libs</directory>
                            <targetPath>WEB-INF/lib/</targetPath>
                            <includes>
                                <include>**/*.jar</include>
                            </includes>
                        </resource>
                    </webResources>
                </configuration>
            </plugin>
```

```xml
<!-- 工程本地依赖的导入 -->
<dependency>
			<groupId>net.sf.json-lib</groupId>
			<artifactId>json-lib</artifactId>
			<version>2.4</version>
			<scope>system</scope>
			<systemPath>${project.basedir}/libs/json-lib-2.4-jdk15.jar</systemPath>
		</dependency>
```



#### maven-surefire-plugin

将某些测试排除或加入打包流程

```xml
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<configuration>
					<includes>
						<includes>**/buid/*Test.java</includes>
					</includes>
					<excludes>
						<exclude>**/unit/*.java</exclude>
					</excludes>
				</configuration>
			</plugin>
```

#### jar包依赖抽离

spring-boot-maven-plugin可以控制不打jar包，但是运行时要在命令里加上外部依赖：

java  -Dloader.path=[out_lib_path]  -jar [the_jar_path.jar]

如果-Dloader不管用，还是用maven-jar-plugin，它会把lib下的jar写入MANIFEST.MF。部署时把lib放同样的相对路径下，正常java -jar启动

```xml
			<plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                	<layout>ZIP</layout>
                	<includes>
                		<include>
                			<groupId>${groupId}</groupId>
                			<artifactId>${artifactId}</artifactId>
                		</include>
                	</includes>
                </configuration>
                <executions>
                	<execution>
                		<goals>
	                		<goal>repackage</goal>
                		</goals>
                	</execution>
                </executions>
            </plugin>
```



```xml
 			<plugin>
			    <groupId>org.apache.maven.plugins</groupId>
			    <artifactId>maven-jar-plugin</artifactId>
			    <configuration>
			        <archive>
			            <manifest>
			                <!--addClasspath表示需要加入到类构建路径-->
			                <addClasspath>true</addClasspath>
			                <!--classpathPrefix指定生成的Manifest文件中Class-Path依赖lib前面都加上路径,构建出lib/xx.jar-->
			                <classpathPrefix>lib/</classpathPrefix>
			                <mainClass>com.gzq.demo.BestDemoApplication</mainClass>
			            </manifest>
			        </archive>
			    </configuration>
			</plugin>
```



#### maven-dependency-plugin

辅助于jar包瘦身，把依赖输出到工程本地文件夹

```xml
			<plugin>
            	<groupId>org.apache.maven.plugins</groupId>
            	<artifactId>maven-dependency-plugin</artifactId>
            	<executions>
            		<execution>
            			<id>copy-dependencies</id>
            			<phase>package</phase>
            			<goals>
            				<goal>copy-dependencies</goal>
            			</goals>
            			<configuration>
            				<outputDirectory>${project.build.directory}/lib</outputDirectory>
            				<excludeTransitive>false</excludeTransitive>
            				<stripVersion>false</stripVersion>
            				<includeScope>runtime</includeScope>
            			</configuration>
            		</execution>
            	</executions>
            </plugin>
```



