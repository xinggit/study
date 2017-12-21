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
  

Spark自定义排序
    源码
    class CustomSort(val a: Int,val b: Int,val c: Int,val d: Int) extends Ordered[CustomSort] with Serializable {
        override def compare(that: CustomSort): Int = {
              println("start---->sortby")
                if (a != that.a) {
                  a - that.a
                } else if (b != that.b) {
                  that.b - b
                } else if (c != that.c) {
                  that.c - c
                } else {
                  that.d - d
             }
        }
        override def toString = s"CustomSort($a, $b, $c, $d)"
  }
  一、自定义排序先继承Ordered这个类型，实现compare这个方法
      val rdd = sc.textFile("D:/1111zx/hs_err_pid32514.log")
      val c = rdd.map(f => {
        val a = f.split("\t");
        new CustomSort(a(0).toInt, a(1).toInt, a(2).toInt, a(3).toInt)
      })
      println("###################################")
      println("###################################")
      val a = c.sortBy(f=>f,false).take(6)
      for(b <- a) {
        println(b)
      }
      println("###################################")
      println("###################################")
  二、将数据包装成自定义的类，sortby这个函数将会自动根据类的compare方法来排序
      
     
Spark自定义分区
  一、先决条件：必须是key-value对的类型
  二、自定义分区类，继承Partitioner类
    源码
      class CustomPartitioner extends Partitioner with Serializable {
      override def numPartitions = 2

      override def getPartition(key: Any): Int = {
        key.asInstanceOf[Int] % numPartitions
      }
    }
  三、调用分区类
          val rdd = sc.textFile("D:/1111zx/hs_err_pid32514.log", 4)
          val vs = rdd.map(f => {
            val ays = f.split("\t");
            (ays(0).toInt, ays(1).toInt)
          }).partitionBy(new CustomPartitioner)
    
      
  四、sql的用法
   1、将两个不同的字段，放在一组
    示例：  group by A.day,
           A.CHNL_CD,
           A.INVEST_TERM,
           CASE WHEN PRODUCT_ID=10 THEN 3 ELSE A.PRODUCT_ID END
        利用CASE WHEN 将产品ID等于10和等于3的放在一组
   2、GROUPING SETS作为GROUP BY的子句，允许开发人员在GROUP BY语句后面指定多个统计选项，可以简单理解为多条group by语句通过union all把查询结果聚合     起来结合起来，下面是几个实例可以帮助我们了解
   关键字1：WITH CUBE
   关键字2：grouping sets
   关键字3：ROLL UP
      
  五  maven打包idea项目，当存在多个目录的时候  
  　　需要说明的是，如果一个maven项目中有多个子目录，每一个子目录中的pom.xml对应一个项目，它的作用范围只有这一个子目录下的。比如扫描配置文件，如果要让一个子目录下的pom.xml扫描另一个子目录下的配置文件，那是做不到的。在打jar包的时候，只运行当前的pom.xml文件。
 
　　当然也有其他的打包方法，比如使用spring-boot-maven-plugin插件在打Jar包时，会引入依赖包。
　　它的pom.xml文件配置为：
复制代码
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <archive>
                    <manifest>
                        <addClasspath>true</addClasspath>
                        <useUniqueVersions>false</useUniqueVersions>
                        <classpathPrefix>lib/</classpathPrefix>
                        <mainClass>cn.mymaven.test.TestMain</mainClass>
                    </manifest>
                    <manifestEntries>
                        <version>${project.version}</version>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
