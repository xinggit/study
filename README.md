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
