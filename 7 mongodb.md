### 常用命令

* 开启mogodb 并指定存储位置

  mongod –dbpath dir 

* 导入数据

  mongoimport –db dbname –collection collectionName –drop –file c:\data\a.json

  –drop 表示删除之前的数据 

* 客户端操作命令

  show dbs 	//显示所有数据库

  use dbname	//使用或者创建一个新的库

  show collections	//显示库里边所有的集合

  db.collectionName.inert(obj) 	//在collectionName集合里插入一条文档，如果集合不存在则创建

  db.collectionName.find()		//查找集合名为collectionName的所有文档记录

  db.collectionName.find({k:v}) //查询一个条件

  db.collectionName.find({k:v,k:v}) //&&查询多个条件

  db.collectionName.find({$or:[{k:v},{k:v}]}) 	//OR查询

  db.collectionName.find(“age”:{$gt:10}) //大于

  db.collectionName.find(“age”:{$lt:10}) //小于

  db.collectionName.find(“age”:{$gt:5,$lt:10}) // 5 < x < 10

  db.collectionName.find().limit(10),skip(2) //跳过签两条后选10条

  db.collectionName.update(       

  ​	{“age”:10},

  ​	{$set:{“age”:100,“name”:“haitao”}},           //修改

  ​	{multi:true} //修改多行，不加这个只修改第一行

  )

  db.collectionName.update(

  ​	{“age”:10},

  ​	{

  ​		“age”:100,“name”:“haitaos”  //替换

  ​	}

  ​	{multi:true} 

  )

  

  删文档

  db.collectionName.remove({“age”:10})  //删除多条

  db.collectionName.remove({“age”:10},{justOne:true}) //只删一条

  db.collectionName.remove({})  //删除集合内所有文档

  db.collectionName.drop()  //删除集合

  db.dropDatabase() //删除当前数据库（要use进去才能删）

  排序：

  db.collectionName.find({“age”:{$lt:10}}).sort({“age”:1},{“name”:-1}) //按age升序 -1降序，如果age一样则按name降序