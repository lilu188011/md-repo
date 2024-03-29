### 											    maven配置

#### 1.配置单个仓库

```
<mirrors>
    <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>

tip: 1.mirrorOf中配置的星号，表示匹配所有的artifacts，也就是所有下载都使用这里的代理地址。上面的mirrorOf配置了具体的名字，指的是repository的名字。
	 2.mirrors里配置多个只有一个生效
```

#### 2.配置多个仓库

##### 2.1配置多个profile

```xml
<profiles>
    <profile>
      <id>aliyun</id> 
      <repositories>
        <repository>
          <id>aliyun</id> 
          <url>http://maven.aliyun.com/nexus/content/groups/public/</url> 
          <releases>
            <enabled>true</enabled>
          </releases> 
          <snapshots>
            <enabled>true</enabled> 
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile> 
    <profile>
      <id>maven-central</id> 
      <repositories>
        <repository>
          <id>maven-central</id> 
          <url>http://central.maven.org/maven2/</url> 
          <releases>
            <enabled>true</enabled>
          </releases> 
          <snapshots>
            <enabled>true</enabled> 
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
    </profile>
<profiles>

```

##### 2.2 激活对应profile

```xml
<activeProfiles>
    <activeProfile>aliyun</activeProfile>
    <activeProfile>maven-central</activeProfile>
</activeProfiles>
```

#### 3.repositories  distributionManagement pluginRepositories

```
repositorie 表示下载项目依赖库文件的maven仓库地址
distributionManagement   表示项目打包成库文件后要上传到仓库地址
pluginRepositories 		pluginRepositories 表示插件的下载仓库地址
```

```
<repositories>
  <repository>
      <!-- 仓库ID -->
      <id>nexus</id>
      <!-- 仓库名称 -->
      <name>Nexus</name>
      <!-- 仓库地址 -->
      <url>http://192.168.1.x:xxxx/repository/maven-public/</url>
      <!-- 仓库中版本为releases的构件 -->
      <releases>
          <!-- 是否支持更新-->
          <enabled>true</enabled>
          <!-- 构件更新的策略，可选值有daily, always, never, interval:X(其中的X是一个数字，表示间隔的时间，单位min)，默认为daily-->
          <updatePolicy>always</updatePolicy>  
          <!-- 校验码异常的策略，可选值有ignore, fail, warn -->
          <checksumPolicy>warn</checksumPolicy>  
      </releases>
       <!-- 仓库版本为snapshots的构件-->
      <snapshots>
           <!-- 是否支持更新-->
          <enabled>true</enabled>  
          <!-- 同上 -->
          <updatePolicy>always</updatePolicy>  
          <!-- 同上 -->
          <checksumPolicy>warn</checksumPolicy>  
      </snapshots>
  </repository>
</repositories>
<distributionManagement>
    <!-- 正式版本 -->
    <repository>
        <uniqueVersion>false</uniqueVersion>
         <!-- nexus服务器中用户名（settings.xml中<server>的id）-->
        <id>releases</id>
         <!-- 自定义名称 -->
        <name>Releases Repository</name>
        <url>http://192.168.1.x:xxxx/repository/maven-releases/</url>
        <layout>default</layout>
    </repository>
    <!-- 快照 -->
    <snapshotRepository>
        <uniqueVersion>true</uniqueVersion>
        <id>snapshots</id>
        <name>Snapshots Repository</name>
        <url>http://192.168.1.x:xxxx/repository/maven-snapshots/</url>
        <layout>legacy</layout>
    </snapshotRepository>
</distributionManagement>
<pluginRepositories>
  <pluginRepository>
      <id>nexus</id>
      <name>Nexus</name>
      <url>http://192.168.1.x:xxxx/repository/maven-public/</url>
      <releases>
          <enabled>true</enabled>
          <updatePolicy>always</updatePolicy>  
          <checksumPolicy>warn</checksumPolicy>  
      </releases>
      <snapshots>
          <enabled>true</enabled>  
          <updatePolicy>always</updatePolicy>  
          <checksumPolicy>warn</checksumPolicy>  
      </snapshots>
  </pluginRepository>
</pluginRepositories>
```

#### 4.使用profiles和properties元素来定义两个不同的构建配置

```xml
<profiles>
       <profile>
           <activation>
              <!-- <os>
                   <family>Windows</family>
               </os>-->
               <activeByDefault>true</activeByDefault>
           </activation>
           <id>local</id>
           <properties>
               <dubbo.registry.address>10.6.1.1:2181</dubbo.registry.address>
               <jdbc.passwod></jdbc.passwod>
           </properties>
       </profile>
       <profile>
           <id>test</id>
           <properties>
               <dubbo.registry.address>10.6.14.11:2181</dubbo.registry.address>
           </properties>
       </profile>
   </profiles>
   
   <build>
       <resources>
           <resource>
               <directory>${project.basedir}/src/main/resources</directory>
               <filtering>true</filtering>
           </resource>
           <resource>
               <directory>${project.basedir}/bin</directory>
               <targetPath>/bin</targetPath>
               <filtering>true</filtering>
           </resource>
       </resources>
   </build>


使用（在xml或properties中使用）

	a) xml文件中使用

  		<dubbo:registry protocol="zookeeper" address="${dubbo.registry.address}"/>

	b) properties文件中使用

		jdbc.password=${jdbc.passwod}
执行maven命令，使profiles的local节点生效

		install -P local -DskipTests


maven profile propoties 也可读取程序参数   https://www.jb51.net/article/270985.htm
```

#### 5.maven依赖理解

```
dependencyManagement 管理依赖版本、只有显示引入才会导入   dependencies 直接导入
maven中scope为import时
	只会导入 dependencyManagement 中的依赖  dependencies 依赖不做导入  
```
#### 6.maven scope
```
compile：默认值 他表示被依赖项目需要参与当前项目的编译，还有后续的测试，运行周期也参与其中，是一个比较强的依赖。打包的时候通常需要包含进去
test：依赖项目仅仅参与测试相关的工作，包括测试代码的编译和执行，不会被打包，例如：junit
runtime：表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与compile相比，跳过了编译而已。例如JDBC驱动，适用运行和测试阶段
provided：打包的时候可以不用包进去，别的设施会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。相当于compile，但是打包阶段做了exclude操作
system：从参与度来说，和provided相同，不过被依赖项不会从maven仓库下载，而是从本地文件系统拿。需要添加systemPath的属性来定义路径

provided,test 不参与依赖传递

```



