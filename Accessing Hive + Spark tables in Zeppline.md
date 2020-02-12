

import com.hortonworks.hwc.HiveWarehouseSession
val hive = HiveWarehouseSession.session(spark).build()
val hiveDf = hive.executeQuery("select * from test_beeline.test")
hiveDf.createOrReplaceTempView("hivetable1")
spark.sql("select * from hivetable1 join spark_test.test on hivetable1.id ==  test.name").show



