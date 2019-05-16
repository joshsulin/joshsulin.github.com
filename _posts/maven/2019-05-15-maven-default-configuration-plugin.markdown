---
layout: post
title:  "Maven提供的默认插件"
date:   2019-05-15 21:12:15 +0800
categories: Maven
tag: maven
---

Maven的生命周期是抽象的，这意味着生命周期本身不做任何实际的工作，在Maven的设计中，实际的任务（如编译源代码）都交由插件来完成。

这篇文章将列出Maven不同生命周期各阶段所配置的默认插件.

在Maven源码中, 有一个配置文件, 记录了 `生命周期--阶段--默认插件` 的映射关系. 同时Maven针对于不同的打包方式(pom、jar、ejb、maven-plugin、war、ear、rar) 设置了默认的 `生命周期(default)--阶段--默认插件` 的映射关系.

关于该配置文件所处位置 `maven安装包所在目录/lib/maven-core-3.5.0.jar!/META-INF/plexus/components.xml` 或者 <a href="/components.xml" target="_blank">在线查看</a>

根据该配置文件的数据, 可以列出各个阶段应该执行哪些插件?

### clean 生命周期

| 阶段       | 默认插件 |
| --------   | :-----   |
| pre-clean  |          |
| clean      | `org.apache.maven.plugins:maven-clean-plugin:2.5:clean` |
| post-clean |          |

### default 生命周期

#### 针对 pom 打包方式

| 阶段                    | pom     |
| --------                | :-----  |
| validate                |         |
| initialize              |         |
| generate-sources        |         |
| process-sources         |         |
| generate-resources      |         |
| process-resources       |         |
| compile                 |         |
| process-classes         |         |
| generate-test-sources   |         |
| process-test-sources    |         |
| generate-test-resources |         |
| process-test-resources  |         |
| test-compile            |         |
| process-test-classes    |         |
| test                    |         |
| prepare-package         |         |
| package                 |         |
| pre-integration-test    |         |
| integration-test        |         |
| post-integration-test   |         |
| verify                  |         |
| install                 | `org.apache.maven.plugins:maven-install-plugin:2.4:install` |
| deploy                  | `org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy`   |

#### 针对 jar 打包方式

| 阶段                    | jar     |
| --------                | :-----  |
| validate                |         |
| initialize              |         |
| generate-sources        |         |
| process-sources         |         |
| generate-resources      |         |
| process-resources       | `org.apache.maven.plugins:maven-resources-plugin:2.6:resources` |
| compile                 | `org.apache.maven.plugins:maven-compiler-plugin:3.1:compile` |
| process-classes         |         |
| generate-test-sources   |         |
| process-test-sources    |         |
| generate-test-resources | `org.apache.maven.plugins:maven-resources-plugin:2.6:testResources` |
| process-test-resources  |         |
| test-compile            | `org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile` |
| process-test-classes    |         |
| test                    | `org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test` |
| prepare-package         |         |
| package                 | `org.apache.maven.plugins:maven-jar-plugin:2.4:jar` |
| pre-integration-test    |         |
| integration-test        |         |
| post-integration-test   |         |
| verify                  |         |
| install                 | `org.apache.maven.plugins:maven-install-plugin:2.4:install` |
| deploy                  | `org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy`   |

#### 针对 war 打包方式

| 阶段                    | war     |
| --------                | :-----  |
| validate                |         |
| initialize              |         |
| generate-sources        |         |
| process-sources         |         |
| generate-resources      |         |
| process-resources       | `org.apache.maven.plugins:maven-resources-plugin:2.6:resources` |
| compile                 | `org.apache.maven.plugins:maven-compiler-plugin:3.1:compile` |
| process-classes         |         |
| generate-test-sources   |         |
| process-test-sources    |         |
| generate-test-resources |         |
| process-test-resources  | `org.apache.maven.plugins:maven-resources-plugin:2.6:testResources` |
| test-compile            | `org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile` |
| process-test-classes    |         |
| test                    | `org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test` |
| prepare-package         |         |
| package                 | `org.apache.maven.plugins:maven-jar-plugin:2.4:war` |
| pre-integration-test    |         |
| integration-test        |         |
| post-integration-test   |         |
| verify                  |         |
| install                 | `org.apache.maven.plugins:maven-install-plugin:2.4:install` |
| deploy                  | `org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy`   |

#### 针对 maven-plugin 打包方式

| 阶段                    | maven-plugin |
| --------                | :-----  |
| validate                |         |
| initialize              |         |
| generate-sources        |         |
| process-sources         |         |
| generate-resources      |         |
| process-resources       | `org.apache.maven.plugins:maven-resources-plugin:2.6:resources` |
| compile                 | `org.apache.maven.plugins:maven-compiler-plugin:3.1:compile` |
| process-classes         | `org.apache.maven.plugins:maven-plugin-plugin:3.2:descriptor` |
| generate-test-sources   |         |
| process-test-sources    |         |
| generate-test-resources |         |
| process-test-resources  | `org.apache.maven.plugins:maven-resources-plugin:2.6:testResources` |
| test-compile            | `org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile` |
| process-test-classes    |         |
| test                    | `org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test` |
| prepare-package         |         |
| package                 | `org.apache.maven.plugins:maven-jar-plugin:2.4:war`、`org.apache.maven.plugins:maven-plugin-plugin:3.2:addPluginArtifactMetadata` |
| pre-integration-test    |         |
| integration-test        |         |
| post-integration-test   |         |
| verify                  |         |
| install                 | `org.apache.maven.plugins:maven-install-plugin:2.4:install` |
| deploy                  | `org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy`   |

以上为常见的打包方式所配置的默认插件, 当然maven还对rar、ejb、ear这些打包方式配置了默认插件, 由于平时不常用, 这里不列出, 请查看配置文件.

### site 生命周期

| 阶段        | 默认插件 |
| --------    | :-----   |
| pre-site    |          |
| site        | `org.apache.maven.plugins:maven-site-plugin:3.3:site` |
| post-site   |          |
| site-deploy | `org.apache.maven.plugins:maven-site-plugin:3.3:deploy` |
