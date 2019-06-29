---
layout:     post
title:      "idea 控制台输出 中文乱码 解决方法"
subtitle:   " Intellij IDEA"
date:       2017-07-13 09:23:28
author:     "TyurinTsien"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - 开发工具
---


# idea 控制台输出 中文乱码 解决方法
+ TyurinTsien
+ 2017年07月21日09:23:28

摘要: intellij idea ULTIMATE 2017.2时，console 会输出中文乱码。下面对maven构建项目以及tomcat（不以maven构建）的出现的这种问题解决。

## tomcat输出到控制台（console）出现中文乱码

设置Run/Debug Configuration中设置environment variables 来解决。

+ Idea=>Run=>Edit Configuration，弹出的对话框中，在Startup/Connection 中Run中添加environment variables
+ JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8.如下图所示：


![图例](../../../../img/posts/Screen Shot 2017-07-21 at 09.19.33.png)

## 对于maven构建的项目
+ 由于idea中maven的配置优先，需要在pom.xml中对maven-surefire-plugin进行配置。

```markdown
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.12.4</version>
        <configuration>
          <forkMode>once</forkMode>
          <argLine>-Dfile.encoding=UTF-8</argLine>
        </configuration>
      </plugin>
    </plugins>
```