---
title: Maven实战
date: 2017-07-29 16:13:32
tags: maven
---
* 构建是什么，build这个不是IDE帮我们完成的吗，为啥需要maven
* 平时用到的maven都是依赖管理工具和项目信息管理工具
* maven有一个坐标系统准确定位每一个构建（artifact）
* 坐标系统就是按package来一级级传递的吧
* 构建工具有 maven ant ide make等
* IDE都是手工的进行鼠标点击来完成编译、测试、代码生成，低效还容易出错。
* make是最早的构建工具，有一个Makefile的脚本文件驱动，在Linux下比较好使。
* Ant意思是another neat tool，构建工具，基本结构也是target和depends，没有依赖管理，可以借助Ivy管理依赖。
* maven是声明式的，有很好的依赖管理。
* maven可以创建内部的仓库服务器，中央仓库会有点慢而且，有些jar不一定有.使用nexus建立自己的maven仓库。
* maven的坐标系统，就是groupid artifactid等
* groupid artifactid version 依赖的基本坐标
* type 依赖的类型，默认为jar
* scope以来的范围，就是作用域
* optional 标记依赖是否可选
* exclusion 用来排除依赖传递性
* 依赖范围就是用来控制依赖与这三种classpath（编译、测试、运行）的关系
* <scope> </scope>有这几种依赖范围，
* compile：编译依赖范围，这也是默认的依赖范围，三个阶段都可以。
* test：测试依赖范围，只对测试依赖有效
* provided：已提供依赖范围，只对编译和测试有效，运行无效。
* runtime：测试和运行时有效
* system：和provided一样，但是这个必须指定依赖的路径。
* import：这个后续补上，没详细说
* jar包冲突，maven有自己的依赖调解原则，第一原则是最近者优先，最先依赖的那个会被使用，如果路径长度是一样的就看依赖在pom文件中的顺序。