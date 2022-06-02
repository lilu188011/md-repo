### 										smart-doc + torna生成文档&管理

#### 一、springboot集成smart-doc

```xml
 <plugin>
                <groupId>com.github.shalousun</groupId>
                <artifactId>smart-doc-maven-plugin</artifactId>
                <version>2.2.7</version>
                <configuration>
                    <!--指定生成文档的使用的配置文件,配置文件放在自己的项目中-->
                    <configFile>./src/main/resources/smart-doc.json</configFile>
                    <!--指定项目名称-->
                    <projectName>测试</projectName>
                    <!--smart-doc实现自动分析依赖树加载第三方依赖的源码，如果一些框架依赖库加载不到导致报错，这时请使用excludes排除掉-->
<!--                    <excludes>-->
<!--                        &lt;!&ndash;格式为：groupId:artifactId;参考如下&ndash;&gt;-->
<!--                        &lt;!&ndash;也可以支持正则式如：com.alibaba:.* &ndash;&gt;-->
<!--                        <exclude>com.alibaba:fastjson</exclude>-->
<!--                    </excludes>-->
                    <!--includes配置用于配置加载外部依赖源码,配置后插件会按照配置项加载外部源代码而不是自动加载所有，因此使用时需要注意-->
                    <!--smart-doc能自动分析依赖树加载所有依赖源码，原则上会影响文档构建效率，因此你可以使用includes来让插件加载你配置的组件-->
<!--                    <includes>-->
<!--                        &lt;!&ndash;格式为：groupId:artifactId;参考如下&ndash;&gt;-->
<!--                        &lt;!&ndash;也可以支持正则式如：com.alibaba:.* &ndash;&gt;-->
<!--                        <include>com.alibaba:fastjson</include>-->
<!--                        &lt;!&ndash; 如果配置了includes的情况下， 使用了mybatis-plus的分页需要include所使用的源码包 &ndash;&gt;-->
<!--                        <include>com.baomidou:mybatis-plus-extension</include>-->
<!--                        &lt;!&ndash; 如果配置了includes的情况下， 使用了jpa的分页需要include所使用的源码包 &ndash;&gt;-->
<!--                        <include>org.springframework.data:spring-data-commons</include>-->
<!--                    </includes>-->
                </configuration>
                <executions>
                    <execution>
                        <!--如果不需要在执行编译时启动smart-doc，则将phase注释掉-->
<!--                        <phase>compile</phase>-->
                        <goals>
                            <!--smart-doc提供了html、openapi、markdown等goal，可按需配置-->
                            <goal>html</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

```
项目地址:  https://gitee.com/xlfast/smart-doc-demo
```

#### 二、安装torna

![image-20220602165944882](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220602165944882.png)

```
docker run --name torna --restart=always \
  -p 7700:7700 \
  -e JAVA_OPTS="-Xms256m -Xmx256m" \
  --link=mysql-master \
  -v /opt/torna/config:/torna/config \
  -d tanghc2020/torna:latest
```

##### /opt/torna/config/appliction.properties如下

```
# 服务端口
server.port=7700

# 数据库连接配置
mysql.host=mysql-master:3306
mysql.username=backup
mysql.password=backup
```

#### 三、 smart-doc推送配置

```json
{
  "serverUrl": "http://127.0.0.1",
  "isStrict": false,
  "outPath": "D:\\torna",
  "packageFilters": "",
  "projectName": "smart-doc",
  "appToken": "2d8e86fdad9e49e3aa9d9029b6e17873",
  "openUrl": "http://192.168.56.102:7700/api",
  "debugEnvName":"测试环境",
  "replace": true,
  "debugEnvUrl":"http://192.168.56.102:7700/debug"
}
```

