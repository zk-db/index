---
title: Spark之基础概念
date: 2022-10-25 15:34:04
tags:
  - spark
  - basic
---

基于内存，分布式计算，函数式处理



# RDD：Resilient Distributed Dataset

弹性分布式数据集：不可变，分布式抽象数据。使用时可以显示的将其换存在内存中以提高速度



- 分片，每一个分片会被一个计算任务处理，可以在创建RDD时指定分片数
- 迭代器，每个分片计算结果可以通过迭代器的compute函数进行整合，无需保存每次计算的结果
- 依赖关系，RDD转换后形成新的RDD，新的RDD就依赖之前的结果。在部分分区计算失败或数据丢失时，可以具体到依赖的分区进行重新计算即可
- 分片函数，哈希分片语范围分片，其决定了分片数量
- 移动计算而不是移动数据，任务调度时尽可能将任务下发到所需处理的数据块存储节点执行



# 创建RDD

- HDFS
- 数据库
- 本地文件
- 其他



# 算子

- Transformation

  将一个RDD转换为另一个RDD，transfromation具有lazy load特性。需要遇到action算子时才会执行

  | **Transformation**                                       | **含义**                                                     |
  | -------------------------------------------------------- | ------------------------------------------------------------ |
  | **map**(func)                                            | 返回一个新的RDD，该RDD由每一个输入元素经过func函数转换后组成 |
  | **filter**(func)                                         | 返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成 |
  | **flatMap**(func)                                        | 类似于map，但是每一个输入元素可以被映射为0或多个输出元素（所以func应该返回一个序列，而不是单一元素） |
  | **mapPartitions**(func)                                  | 类似于map，但独立地在RDD的每一个分片上运行，因此在类型为T的RDD上运行时，func的函数类型必须是Iterator[T] => Iterator[U] |
  | **mapPartitionsWithIndex**(func)                         | 类似于mapPartitions，但func带有一个整数参数表示分片的索引值，因此在类型为T的RDD上运行时，func的函数类型必须是(Int, Interator[T]) => Iterator[U] |
  | **sample**(withReplacement, fraction, seed)              | 根据fraction指定的比例对数据进行采样，可以选择是否使用随机数进行替换，seed用于指定随机数生成器种子 |
  | **union**(otherDataset)                                  | 对源RDD和参数RDD求并集后返回一个新的RDD                      |
  | **intersection**(otherDataset)                           | 对源RDD和参数RDD求交集后返回一个新的RDD                      |
  | **distinct**([numTasks]))                                | 对源RDD进行去重后返回一个新的RDD                             |
  | **groupByKey**([numTasks])                               | 在一个(K,V)的RDD上调用，返回一个(K, Iterator[V])的RDD        |
  | **reduceByKey**(func, [numTasks])                        | 在一个(K,V)的RDD上调用，返回一个(K,V)的RDD，使用指定的reduce函数，将相同key的值聚合到一起，与groupByKey类似，reduce任务的个数可以通过第二个可选的参数来设置 |
  | **aggregateByKey**(zeroValue)(seqOp, combOp, [numTasks]) | 先按分区聚合 再总的聚合  每次要跟初始值交流 例如：aggregateByKey(0)(_+_,_+_) 对k/y的RDD进行操作 |
  | **sortByKey**([ascending], [numTasks])                   | 在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD |
  | **sortBy**(func,[ascending], [numTasks])                 | 与sortByKey类似，但是更灵活 第一个参数是根据什么排序 第二个是怎么排序 false倒序  第三个排序后分区数 默认与原RDD一样 |
  | **join**(otherDataset, [numTasks])                       | 在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD 相当于内连接（求交集） |
  | **cogroup**(otherDataset, [numTasks])                    | 在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的RDD |
  | **cartesian**(otherDataset)                              | 两个RDD的笛卡尔积 的成很多个K/V                              |
  | **pipe**(command, [envVars])                             | 调用外部程序                                                 |
  | **coalesce**(numPartitions**)**                          | 重新分区 第一个参数是要分多少区，第二个参数是否shuffle 默认false 少分区变多分区 true  多分区变少分区 false |
  | **repartition**(numPartitions)                           | 重新分区 必须shuffle 参数是要分多少区 少变多                 |
  | **repartitionAndSortWithinPartitions**(partitioner)      | 重新分区+排序 比先分区再排序效率高 对K/V的RDD进行操作        |
  | **foldByKey**(zeroValue)(seqOp)                          | 该函数用于K/V做折叠，合并处理 ，与aggregate类似  第一个括号的参数应用于每个V值 第二括号函数是聚合例如：_+_ |
  | **combineByKey**                                         | 合并相同的key的值 rdd1.combineByKey(x => x, (a: Int, b: Int) => a + b, (m: Int, n: Int) => m + n) |
  | **partitionBy****（partitioner）**                       | 对RDD进行分区 partitioner是分区器 例如new HashPartition(2    |
  | **cache**                                                | RDD缓存，可以避免重复计算从而减少时间，区别：cache内部调用了persist算子，cache默认就一个缓存级别MEMORY-ONLY ，而persist则可以选择缓存级别 |
  | **persist**                                              |                                                              |
  |                                                          |                                                              |
  | **Subtract****（rdd）**                                  | 返回前rdd元素不在后rdd的rdd                                  |
  | **leftOuterJoin**                                        | leftOuterJoin类似于SQL中的左外关联left outer join，返回结果以前面的RDD为主，关联不上的记录为空。只能用于两个RDD之间的关联，如果要多个RDD关联，多关联几次即可。 |
  | **rightOuterJoin**                                       | rightOuterJoin类似于SQL中的有外关联right outer join，返回结果以参数中的RDD为主，关联不上的记录为空。只能用于两个RDD之间的关联，如果要多个RDD关联，多关联几次即可 |
  | subtractByKey                                            | substractByKey和基本转换操作中的subtract类似只不过这里是针对K的，返回在主RDD中出现，并且不在otherRDD中出现的元素 |

- Action

  触发代码执行，一段spark代码至少需要一个action

  | **Action**                                        | **含义**                                                     |
  | ------------------------------------------------- | ------------------------------------------------------------ |
  | **reduce**(*func*)                                | 通过func函数聚集RDD中的所有元素，这个功能必须是课交换且可并联的 |
  | **collect**()                                     | 在驱动程序中，以数组的形式返回数据集的所有元素               |
  | **count**()                                       | 返回RDD的元素个数                                            |
  | **first**()                                       | 返回RDD的第一个元素（类似于take(1)）                         |
  | **take**(*n*)                                     | 返回一个由数据集的前n个元素组成的数组                        |
  | **takeSample**(*withReplacement*,*num*, [*seed*]) | 返回一个数组，该数组由从数据集中随机采样的num个元素组成，可以选择是否用随机数替换不足的部分，seed用于指定随机数生成器种子 |
  | **takeOrdered**(*n*, *[ordering]*)                |                                                              |
  | **saveAsTextFile**(*path*)                        | 将数据集的元素以textfile的形式保存到HDFS文件系统或者其他支持的文件系统，对于每个元素，Spark将会调用toString方法，将它装换为文件中的文本 |
  | **saveAsSequenceFile**(*path*)                    | 将数据集中的元素以Hadoop sequencefile的格式保存到指定的目录下，可以使HDFS或者其他Hadoop支持的文件系统。 |
  | **saveAsObjectFile**(*path*)                      |                                                              |
  | **countByKey**()                                  | 针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数。 |
  | **foreach**(*func*)                               | 在数据集的每一个元素上，运行函数func进行更新。               |
  | **aggregate**                                     | 先对分区进行操作，在总体操作                                 |
  | **reduceByKeyLocally**                            |                                                              |
  | **lookup**                                        |                                                              |
  | **top**                                           |                                                              |
  | **fold**                                          |                                                              |
  | **foreachPartition**                              |                                                              |

# Spark任务基础概念

- Application

  driver程序以及集群执行的代码

- driver

  创建sc，加载数据集，处理数据以及展示流程

- 集群节点

  driver：创建sc，启动spark级集群计算任务

  master：master节点

  worker：集群任务执行节点，启动executor进程

  Executor：执行应用程序以及汇报执行状态

  cluster manager：集群资源管理器，负责申请资源调度任务，如yarn

- jobs

  RDD中的action，每个action变为一个job，然后DAGScheduler回分解stage执行

- stage

  一个job拆分为多组task，每组是一个stage

- task

  送到executor执行的工作单元，两类：shuffleMapTask：输出shffle所需要的数据；resultTask：无需shuffle直接返回处理结果

- partition

  数据划分

- cores

  worker执行进程

- memory

  内存设置

- shuffle

  stage之间的数据拷贝

# 窄依赖与宽依赖

窄依赖：每个RDD只会被一个子RDD所依赖，例如map、filter、union等操作都会产生窄依赖；（独生子女）

宽依赖：每个RDD被多个子RDD所依赖，例如groupByKey、reduceByKey、sortByKey等操作都会产生宽依赖；（超生）



关于join的依赖关系：

两个RDD在进行join操作时，一个RDD的partition仅仅和另一个RDD中已知个数的Partition进行join，那么这种类型的join操作就是窄依赖。否则就是宽依赖。



# DAG图与宽窄依赖

DAGScheduler遇到action job时，会根据宽窄依赖决定stage划分。如果遇到窄依赖会将该action加入当前stage，遇到宽依赖则创建新的stage去执行
