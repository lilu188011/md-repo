## maven-assembly-plugin使用

### 一、插件配置

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.0.2</version>
                <configuration>
                    <excludes>
                        <exclude>*.properties</exclude>
                        <exclude>*.xml</exclude>
                        <exclude>assembly/**</exclude>
                        <exclude>config/**</exclude>
                        <exclude>script/**</exclude>
                        <exclude>cert/**</exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <excludes>
                    	<!-- maven test 跳过指定java类测试  -->
                        <exclude>**/*IT.java</exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <executions>
                    <execution>
                        <id>make-zip</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <appendAssemblyId>false</appendAssemblyId>
                            <descriptors>
                                <descriptor>src/main/resources/assembly/zip.xml</descriptor>
                            </descriptors>
                        </configuration>
                    </execution>
                </executions>
                <configuration>
                    <archiverConfig>
                        <defaultDirectoryMode>0755</defaultDirectoryMode>
                    </archiverConfig>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### 二 、assembly xml配置文件 zip.xml

```xml
<assembly
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0"
        xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">
    <id>release</id>
    <formats>
        <format>zip</format>
    </formats>
    <baseDirectory>${project.artifactId}</baseDirectory>
    <fileSets>
        <!-- 环境配置优先 -->
        <fileSet>
            <directory>src/main/resources/config/${env}</directory>
            <includes>
                <include>*.xml</include>
                <include>*.properties</include>
                <include>*.conf</include>
            </includes>
            <outputDirectory>conf</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/resources</directory>
            <includes>
                <include>*.xml</include>
                <include>*.properties</include>
                <include>*.conf</include>
                <include>*.yml</include>
                <include>*.yaml</include>
            </includes>
            <outputDirectory>conf</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/resources/config</directory>
            <outputDirectory>support-files</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/resources/script</directory>
            <outputDirectory>bin</outputDirectory>
            <fileMode>0755</fileMode>
        </fileSet>

    </fileSets>
    <!--指定非指定非java类包的位置（如：dll，so）-->
   <!-- <files>
        <file>
            <source>src/main/resources/assembly/otherSys.so</source>
            <outputDirectory>lib</outputDirectory>
            <fileMode>0755</fileMode>
        </file>
    </files>-->

    <dependencySets>
        <dependencySet>
            <useProjectArtifact>true</useProjectArtifact>
            <outputDirectory>lib</outputDirectory>
            <scope>runtime</scope>
            <fileMode>0755</fileMode>
        </dependencySet>
    </dependencySets>
</assembly>
```

目录结构如下

![image-20220629162322720](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220629162323553.png)

三、代码详见

```
https://gitee.com/xlfast/activiti-demo
```

