# Spark

## 常用命令

| 命令             | 参数                     | 示例                                                         | 说明                                                         |
| ---------------- | ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| bin/spark-shell  |                          | bin/spark-shell                                              | 进入shell                                                    |
| bin/spark-submit |                          | bin/spark-submit \ --class org.apache.spark.examples.SparkPi \ --master local[2] \ ./examples/jars/spark-examples_2.12-3.0.0.jar \ 10 | 启动任务                                                     |
|                  | --class                  |                                                              | Spark 程序中包含主函数的类                                   |
|                  | --master                 | 模式：local[*]、spark://linux1:7077、 Yarn                   | Spark 程序运行的模式(环境)                                   |
|                  | --executor-memory 1G     |                                                              | 指定每个 executor 可用内存为 1G                              |
|                  | --total-executor-cores 2 |                                                              | 指定所有executor使用的cpu核数为 2 个                         |
|                  | --executor-cores         |                                                              | 指定每个executor使用的cpu核数                                |
|                  | application-jar          |                                                              | 打包好的应用 jar，包含依赖。这 个 URL 在集群中全局可见。 比 如 hdfs:// 共享存储系统，如果是file:// path，那么所有的节点的 path 都包含同样的 jar |
|                  | application-arguments    |                                                              | 传给 main()方法的参数                                        |
|                  | 10                       |                                                              | 数字 10 表示程序的入口参数，用于设定当前应用的任务数量       |

