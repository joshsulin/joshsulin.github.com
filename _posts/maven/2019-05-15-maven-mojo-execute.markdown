---
layout: post
title:  "运行具体构建任务时, 由哪些插件来完成些构建任务?"
date:   2019-05-15 22:45:15 +0800
categories: Maven
tag: Maven
---

通过<a href="/maven/2019/05/15/maven-default-configuration-plugin/" target="_blank">这篇博客</a>可知, Maven服务启动时, 系统就己经配置了大量的插件, 足以完成项目编译、打包、测试、部署等基本操作.

但是对于生产项目, 总是会添加一些插件, 并且根据功能需求来决定添加的插件是否应该绑定到生命周期中某个阶段还是单独运行.

`这篇博客将说明, 针对采用Maven来进行构建的Java项目, 运行构建任务(mvn xxx), 到底由哪些插件来完成此构建任务?`

通过阅读Maven源码, 对Maven执行逻辑进行总结, 提取出以下 五步 来说明上面的问题.

###  第一步: 获取项目打包方式

```java
// 每个Maven项目都有一个pom.xml文件
......
<groupId>com.josh</groupId>
<artifactId>XXXXXX</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>jar</packaging>
......
```

<packaging></packaging>中配置的信息, 为该Maven项目设定的打包方式, 默认为jar

### 第二步: 根据项目的打包方式, 获取默认配置的maven插件

```java
// maven-core-3.5.0-sources.jar!/org/apache/maven/model/plugin/DefaultLifecycleBindingsInjector.java
Collection<Plugin> defaultPlugins = lifecycle.getPluginsBoundByDefaultToAllLifecycles( packaging );
```
根据Maven默认插件配置数据, 很容易完成.

### 第三步: 获取项目pom.xml文件中配置的插件

```java
<build>
    <plugins>
        <plugin>
            ......
            ......
            ......
        </plugin>
        <plugin>
            ......
            ......
            ......
        </plugin>
        <plugin>
            ......
            ......
            ......
        </plugin>
    <plugins>
</build>
```

### 第四步: 将上面第三步获取的插件和第四步获取的插件进行合并

```java
// 获取Maven默认配置的插件(第二步获取的插件)
List<Plugin> src = source.getPlugins();

if ( !src.isEmpty() )
{
    // 获取项目pom.xml配置的插件(第三步获取的插件)
    List<Plugin> tgt = target.getPlugins();
    
    // 合并后的插件存储到该map中
    Map<Object, Plugin> merged = new LinkedHashMap<>( ( src.size() + tgt.size() ) * 2 );
    
    // 先将项目中的插件放入merged
    for ( Plugin element : tgt )
    {
        Object key = getPluginKey( element );
        merged.put( key, element );
    }
    
    for ( Plugin element : src )
    {
        Object key = getPluginKey( element );
        Plugin existing = merged.get( key );
        if ( existing != null )
        {
            // 当项目中己有插件时, 保留项目中配置的插件版本
            mergePlugin( existing, element, sourceDominant, context );
        }
        else
        {
            merged.put( key, element );
            unmanaged.put( key, element );
        }
    }
}
```

### 第五步: 根据第四步得到的数据, 结合当前构建任务(mvn xxxx), 从中筛选出完成该构建任务需要的插件.

```java
final List<MojoExecution> executions = calculateMojoExecutions( session, project, tasks );

if ( setup )
{
    setupMojoExecutions( session, project, executions );
}
```

#### 5.1: 获取完成指定任务需要依次执行哪些阶段?

```java
Map<String, Map<Integer, List<MojoExecution>>> mappings = new LinkedHashMap<>();
            
for ( String phase : lifecycle.getPhases() ) {
    Map<Integer, List<MojoExecution>> phaseBindings = new TreeMap<>();
    mappings.put( phase, phaseBindings );
    if ( phase.equals( lifecyclePhase ) )
    {
        break;
    }
}
```

#### 5.2: 遍历所有插件, 将不满足条件的插件过滤掉.(条件1、插件对应的生命周期阶段不在任务所对应的生命周期阶段中. 条件2、插件本身没有绑定到生命周期阶段中) 

```java
for ( Plugin plugin : project.getBuild().getPlugins() )
{
    // 当插件没有配置 <executions> 的, 将过滤掉
    for ( PluginExecution execution : plugin.getExecutions() )
    {
        if ( execution.getPhase() != null )
        {
            // 配置了阶段 <phase>compile</phase>
            Map<Integer, List<MojoExecution>> phaseBindings = mappings.get( execution.getPhase() );

            // 如果不存在, 也会被过滤掉, 比如 mvn install, 但是根据第四的逻辑, clean对应的插件也在该项目所属的插件里面, 此步能过滤掉不属于待执行任务所对应的阶段
            if ( phaseBindings != null )
            {
                for ( String goal : execution.getGoals() )
                {
                    MojoExecution mojoExecution = new MojoExecution( plugin, goal, execution.getId() );
                    mojoExecution.setLifecyclePhase( execution.getPhase() );
                    addMojoExecution( phaseBindings, mojoExecution, execution.getPriority() );
                }
            }
        }
        // if not then i need to grab the mojo descriptor and look at the phase that is specified
        // 如果没有那么需要抓住mojo描述符并查看指定的阶段
        // <executions>
        //    <execution>
        //      <id>repackage</id>
        //      <goals>
        //          <goal>repackage</goal>
        //      </goals>
        //    </execution>
        // </executions>
        // mojo描述符(实现插件时, 需要用@Mojo注解标注)如下:
        // @Mojo(name = "repackage", defaultPhase = LifecyclePhase.PACKAGE, requiresProject = true,
        //    threadSafe = true,
        //    requiresDependencyResolution = ResolutionScope.COMPILE_PLUS_RUNTIME,
        //    requiresDependencyCollection = ResolutionScope.COMPILE_PLUS_RUNTIME)
        else
        {
            for (String goal : execution.getGoals())
            {
                MojoDescriptor mojoDescriptor =
                            pluginManager.getMojoDescriptor( plugin, goal, project.getRemotePluginRepositories(),
                                                             session.getRepositorySession() );

                Map<Integer, List<MojoExecution>> phaseBindings = mappings.get( mojoDescriptor.getPhase() );
                if ( phaseBindings != null )
                {
                    MojoExecution mojoExecution = new MojoExecution( mojoDescriptor, execution.getId() );
                    mojoExecution.setLifecyclePhase( mojoDescriptor.getPhase() );
                    addMojoExecution( phaseBindings, mojoExecution, execution.getPriority() );
                }
            }
        }
    }
}
```

最终 addMojoExecution() 方法保存了 运行构建任务(mvn xxx), 满足要求的插件.
