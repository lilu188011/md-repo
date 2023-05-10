## springboot自动提示元数据就在META-INF中的spring-configuration-metadata.json或additional-spring-configuration-metadata.json。
```
    {
      "groups": [],
      "properties": [
        {
          "name": "spring.devtools.add-properties",
          "type": "java.lang.Boolean",
          "description": "Whether to enable development property defaults.",
          "defaultValue": true
        }
      ],
      "hints": []
  }
  
  properties：为设置的提示属性，name为属性名称，type为属性类型，defaultValue为该属性默认值，description为属性描述。
  groups：为properties提供了一个有上下文关联的分组。本身并不指定一个值。
  hints：为属性设置多个提示值。
  https://docs.spring.io/spring-boot/docs/current/reference/html/configuration-metadata.html
```