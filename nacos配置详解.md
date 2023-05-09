#### 1. Nacos对配置的默认理念

```
namespace区分环境：开发环境、测试环境、预发布环境、生产产环境。
group区分不同应用：同个环境内，不同应用的配置，通过group来区分。
主配置是应用专有的配置
因此，主配置应当在dataId上要区分，同时最好还要有group的区分，因为group区分应⽤（虽然dataId上区分了，不设置group也能按应用单独加载）
```

#### 2.nacos默认价值配置文件规则<参考官网>

```
${prefix}-${spring.profile.active}.${file-extension}
${prefix}:
    默认为所属工程配置spring.application.name的值(这就是为什么平时我们直接用服务名就可以),也可以用spring.cloud.nacos.config.prefix来配置.

${spring.profile.active}:
            spring.profile.active即为当前环境对应的profile.注意:当spring.profile.active为空的时候,对应的连接符 - 也将不存在,dataId的拼接格式									变成prefix.{file-extension}

${file-extension}:
			为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型

当配置中心找不到时、才会读取本地配置文件、nacos配置大于本地
```

#### 3.共享配置(shared-configs)和扩展配(extension-config)

```
日常开发中，多个模块可能会有很多共用的配置，比如数据库连接信息，Redis 连接信息，RabbitMQ 连接信息，监控配置等等。那么此时，我们就希望可以加载多个配置，多个项目共享同一个配置之类等功能，Nacos Config 也确实支持。

Nacos在配置路径spring.cloud.nacos.config.extension-config下，允许我们指定⼀个或多个额外配置。

Nacos在配置路径spring.cloud.nacos.config.shared-configs下，允许我们指定⼀个或多个共享配置。

上述两类配置都持三个属性：data-id、group(默认为字符串DEFAULT_GROUP)、refresh(默认为true)。

要在各应用之间共享多个个配置，请使  shared-configs
因此按该理念，shared-configs指定的配置，本来应该是不指定group的，也就是应当归DEFAULT_GROUP这个公共分组。
```

