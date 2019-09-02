## ODP总结

* 单独运行odp脚本

  需要先加载容器

  ~~~php
  $objApplication = Bd_Init::init();  //加载容器
  //$objResponse = $objApplication->bootstrap()->run();  引导运行ap框架
  $dbCfg = "annotate/offEditzuoye"; //数据库配置
  $db = Bd_Db_ConnMgr::getConn($dbCfg);
  $info = $db->select("res_user",array("*"), array("id="=>1));
  var_dump($info);
  ~~~

  