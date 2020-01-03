# Flink install & config & run

## Flink install and config

Flink 是一个以 Java 及 Scala 作为开发语言的开源大数据项目，代码开源在 GitHub 上，并使用 Maven 来编译和构建项目。对于大部分使用 Flink 的同学来说，Java、Maven 和 Git 这三个工具是必不可少的，另外一个强大的 IDE 有助于我们更快的阅读代码、开发新功能以及修复 Bug.

| tools | mark |
|--|:--|
| Java | JDK 8 ( min version ). recommend : Java 8u51 |
| Maven | Maven 3 ( min version ). recommend : Maven 3.2.5 |
| Git | Flink code repository : https://github.com/apache/flink | 


## Compile Code

在我们配置好之前的几个工具后，编译 Flink 就非常简单了，执行如下命令即可：

```shell

mvn clean install -DskipTests
# 或者
mvn clean package -DskipTests

```
常用编译参数

```

-Dfast    主要是忽略QA plugins和JavaDocs的编译
-Dhadoop.version=2.6.1    指定hadoop版本
--settings=${maven_file_path}    显式指定maven settings.xml配置文件

```

编译后

|版本	|注释|
|--|--|
|flink-1.5.1.tar.gz|	Binary 的压缩包|
|flink-1.5.1-bin/flink-1.5.1|	解压后的 Flink binary 目录|
|flink-dist_2.11-1.5.1.jar|	包含 Flink 核心功能的 jar 包|


> note 
>注意： 
国内用户在编译时可能遇到编译失败“Build Failure”（且有 MapR 相关报错），一般都和 MapR 相关依赖的下载失败有关，即使使用了推荐的 settings.xml 配置（其中 Aliyun Maven 源专门为 MapR 相关依赖做了代理），还是可能出现下载失败的情况。问题主要和 MapR 的 Jar 包比较大有关。遇到这些问题时，重试即可。在重试之前，要先根据失败信息删除 Maven local repository 中对应的目录，否则需要等待 Maven 下载的超时时间才能再次出发下载依赖到本地。


- IDE Recommend : IntelliJ IDEA.

## Flink run

 Task 是 Flink 中资源调度的最小单位。

 [Flink DAG](https://ververica.cn/wp-content/uploads/2019/08/2.png)


- Flink 实际运行时包括两类进程：

JobManager（又称为 JobMaster）：协调 Task 的分布式执行，包括调度 Task、协调创 Checkpoint 以及当 Job failover 时协调各个 Task 从 Checkpoint 恢复等。

TaskManager（又称为 Worker）：执行 Dataflow 中的 Tasks，包括内存 Buffer 的分配、Data Stream 的传递等。


Task Slot 是一个 TaskManager 中的最小资源分配单位，一个 TaskManager 中有多少个 Task Slot 就意味着能支持多少并发的 Task 处理。需要注意的是，一个 Task Slot 中可以执行多个 Operator，一般这些 Operator 是能被 Chain 在一起处理的。

## 运行环境准备

准备 Flink binary

- 直接从 Flink 官网上下载 Flink binary 的压缩包

- 或者从 Flink 源码编译而来

安装 Java，并配置 JAVA_HOME 环境变量


## 单机 Standalone 的方式运行 Flink

### 基本的启动流程
最简单的运行 Flink 应用的方法就是以单机 Standalone 的方式运行。

启动集群：

```shell
./bin/start-cluster.sh
```

打开 http://127.0.0.1:8081/ 就能看到 Flink 的 Web 界面。尝试提交 Word Count 任务：

```
./bin/flink run examples/streaming/WordCount.jar
```


大家可以自行探索 Web 界面中展示的信息，比如，我们可以看看 TaskManager 的 stdout 日志，就可以看到 Word Count 的计算结果。

我们还可以尝试通过“–input”参数指定我们自己的本地文件作为输入，然后执行：

```
./bin/flink run examples/streaming/WordCount.jar --input ${your_source_file}
```

停止集群：

```shell
./bin/stop-cluster.sh
```

### 常用配置介绍

* conf / slaves *

conf / slaves 用于配置 TaskManager 的部署，默认配置下只会启动一个 TaskManager 进程，如果想增加一个 TaskManager 进程的，只需要文件中追加一行“localhost”。

也可以直接通过“ ./bin/taskmanager.sh start ”这个命令来追加一个新的 TaskManager：

```shell
./bin/taskmanager.sh start|start-foreground|stop|stop-all
```

* conf/flink-conf.yaml *

conf/flink-conf.yaml 用于配置 JM 和 TM 的运行参数，常用配置有：

```
# The heap size for the JobManager JVM
jobmanager.heap.mb: 1024

# The heap size for the TaskManager JVM
taskmanager.heap.mb: 1024

# The number of task slots that each TaskManager offers. Each slot runs one parallel pipeline.
taskmanager.numberOfTaskSlots: 4

# the managed memory size for each task manager.
```


















