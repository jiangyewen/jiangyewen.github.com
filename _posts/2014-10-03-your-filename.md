---
published: false
---

通过PHP读写MongoDB需要做以下两方面的事情：
- 安装php的mongo插件（php mongo driver）
- 解析名字服务器的字典并连接

### 1. 安装php的mongo拓展插件

-  下载源码并解压

源码包地址 http://pecl.php.net/get/mongo-1.4.0.tgz

-  安装

解压，进入目录，执行：
```shell
phpize
./configure
make &&　make install
```

根据成功的提示，找mongo.so所在路径
```shell
Installing shared extensions: /usr/local/lib/php/extensions/no-debug-non-zts-20090626/
```

-  修改php配置

在php.ini中加入
```shell
extension=/usr/local/lib/php/extensions/no-debug-non-zts-20090626/mongo.so
```
-  重启webserver

-  测试mongo插件是否生效
测试的php文件如下:

```php
$user = array(  
                'first_name' => 'MongoDB',  
                'last_name' => 'Fan',  
                'tags' => array('developer','user')  
             );  
// Configuration  
$dbhost = '10.169.131.79:10001';  
$dbname = 'test';  
// Connect to test database  
$m = new Mongo("mongodb://$dbhost");  
$db = $m->$dbname;  
// Get the users collection  
$users = $db->users;  
//Insert this new document into the users collection  
$res = $users->save($user);  
var_dump($res);  
$data = $users->findOne();  
var_dump($data);  
```



###2. 解析字典服务并与MongoDB连接
```php
<?php
require("/usr/local/zk_agent/names/nameapi.php");

//set the variables with the info we provide
$domain = "signup.mongo.com";
$dbname = "signup";
$collection_name = "sports"; 
$set_name = "shard20004";

//convert domain dict to host
$host = "";
getValueByKey($domain, $host);
if ($host == ""){
    echo "Failed:convert '$domain' dict to host";
    exit;
}

// Connect to database  
$m = new Mongo("mongodb://$host", array("replicaSet" => "$set_name"));
$db = $m->$dbname;


// Get collection  
$collection = $db->$collection_name;

//Insert documents into  collection  
$test_data = array(
                'first_name' => 'MongoDBddd',
                'last_name' => 'Fanssss',
             );
$res = $collection->save($test_data);

//to see if the insertion succeeded
var_dump($res);
$data = $collection->findOne();
var_dump($data);
?>
```
