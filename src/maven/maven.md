##### 生成项目骨架
 > 使用archetype创建项目：mvn archetype:generate

>生成骨架
```
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-archetype-plugin</artifactId>
            <version>3.0.0</version>
         </plugin>
```

> mvn archetype:create-from-project

>mvn clean install（目录：target/generated-sources/archetype）