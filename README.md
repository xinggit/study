一、RDD的各种操作记录

  1、sample
   源码：
     def sample(
      withReplacement: Boolean,
      fraction: Double,
      seed: Long = Utils.random.nextLong): RDD[T] = {.....}
  使用说明：
  参数
      withReplacement：表示是抽出的数据是否放回，true为有放回的抽样，false为无放回的抽样
      fraction：抽出的数据百分比
      seed：随机种子值
      
  示例：
      val rdd = sc.parallelize(1 to 10)
      val sample1 = rdd.sample(true,0.5,3)
      从RDD中随机且有放回的抽出50%的数据，随机种子值为3（即可能以1 2 3的其中一个起始值）
      
  2、randomSplit
  源码：  
  def randomSplit(
      weights: Array[Double],
      seed: Long = Utils.random.nextLong): Array[RDD[T]] = {......}
  使用说明：
  参数
      weights：权重，该函数根据weights权重，将一个RDD切分成多个RDD。
      seed：随机种子值
      
  示例：
      val rdd = sc.parallelize(1 to 10)
      val sample1 = rdd.randomSplit(Array(1.0,5.0),1)
      从rdd中取出数据，将其变为两个rdd数组

  3、sortBy
  源码：  
  def sortBy[K](
      f: (T) => K,
      ascending: Boolean = true,
      numPartitions: Int = this.partitions.length)
      (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T] = withScope {......}
  参数
      f: (T) => K：取值函数，根据该值排序
      ascending：排序方式，true：升序，false：降序
  示例
      val rdd = sc.parallelize(1 to 10)
      rdd.sortBy(f=>f,true)
      
  4、intersection 
  源码
      def intersection(other: RDD[T]): RDD[T] = withScope {......}
  求交集
  
  5、cartesian
  源码
      def cartesian[U: ClassTag](other: RDD[U]): RDD[(T, U)] = withScope {......}
  笛卡尔积
  
  6、glom
  源码
      def glom(): RDD[Array[T]]
      该函数是将RDD中每一个分区中类型为T的元素转换成Array[T]，这样每一个分区就只有一个数组元素。
  示例
      var rdd = sc.makeRDD(1 to 10,3)
      rdd.partitions.size
      res33: Int = 3  //该RDD有3个分区
      rdd.glom().collect
      Array[Array[Int]] = Array(Array(1, 2, 3), Array(4, 5, 6), Array(7, 8, 9, 10))
      //glom将每个分区中的元素放到一个数组中，这样，结果就变成了3个数组
  
      
      
      
      
      
      
      
      
      
      
