1. ### new static()  new self()区别

   * 如果不存在继承关系是两者没有区别

   * 有继承关系时

     new static()返回当前类的实例化对象（具体的new static方法可能在父类中，但调用该方法在子类中），new self()返回该方法实现的具体类的实例。通常情况用 new static(…params) 而不用new self()

2. 单例模式实现

   分析：php的单例模式要满足几个条件

   * 外部通过静态方法调用，外部不能通过new的方式实例化
   * 不能有克隆clone操作（php的对象是引用传递的，如果实现复制要用clone方法）

   ~~~ php
   class Singleton {
       public static $instance;
       public $name;
       //限制不能外部new实例化
       private function __construct(){
       }
       //限制不可克隆，如果外部有clone操作，所有属性clone完成之后会实例会再调用__clone方法。
       private function __clone(){
       }
       public static function getInstance(){
           if (!self::$instance instanceof self) {
               self::$instance = new static();
           }
           return self::$instance;
       }
   }
   
   ~~~

3. assert 断言

   编写代码时，我们总是会做出一些假设，断言就是用于在代码中捕捉这些假设，可以将断言看作是异常处理的一种高级形式。**程序员断言在程序中的某个特定点该的表达式值为真。如果该表达式为假，就中断操作**。
   可以在任何时候启用和禁用断言验证，因此可以在测试时启用断言，而在部署时禁用断言。同样，程序投入运行后，最终用户在遇到问题时可以重新起用断言。
   使用断言可以创建更稳定，品质更好且不易于出错的代码。单元测试必须使用断言！

   ~~~php
   /*
   assert_options
   'ASSERT_ACTIVE=1'  Assert函数的开关
   'ASSERT_WARNING =1'  当表达式为false时，是否要输出警告性的错误提示,issue a PHP warning for each failed assertion
   'ASSERT_BAIL= 0'  是否要中止运行；terminate execution on failed assertions
   'ASSERT_QUIET_EVAL= 0'  是否关闭错误提示，在执行表达式时；disable error_reporting during assertion expression evaluation
   'ASSERT_CALLBACK= (NULL)'  是否启动回调函数 user function to call on failed assertions
   */
   // Active assert and make it quiet
   assert_options(ASSERT_ACTIVE, 1);
   assert_options(ASSERT_WARNING, 0);
   assert_options(ASSERT_QUIET_EVAL, 1);
   
   // Create a handler function
   function my_assert_handler($file, $line, $code)
   {
       echo "<hr>Assertion Failed:File '$file'<br />Line '$line'<br />Code '$code'<br /><hr />";
   }
   // Set up the callback
   assert_options(ASSERT_CALLBACK, 'my_assert_handler');
   // Make an assertion that should fail
   assert('1==2');
   ~~~

4. `call_user_func()`  `call_user_func_array()` `array_map()`

   ~~~php
   //call_user_func()中不可以使用引用传递，解决办法
   $obj = "obj";
   call_user_func(function($o) use (&obj) {
       obj = dosomething($o);
   },$obj); //可以传递多个参数，相应function(...params)
   
   //call_user_func_array() 相对call_user_func()使用场景更多更加灵活(可以传引用)，第一个参数是指定的方法（也可以指定为类对象中的方法），第二个参数必须是数组(可以是多个元素，相应function也要接受多个参数)
   class Obj {
       public function doit(&a) {  //该方法必须是public,或者是 public static
           return $a++;
       }
   }
   $obj = new Obj();
   $a = 10;
   call_user_func_array([$obj,"doit"],[&$a]); //也可以直接用类名 call_user_func_array(["Obj","doit"],[&$a])
   echo $a;//11
   
   //array_map() 返回一个新的数组， 同样可以接受多个参数，每个参数必须是数组
   $a1 = [1,3,5];
   $a2 = [2,4,5];
   $newarr = array_map(function($s1,$s2){
       if(s1 == s2){
           return 1;
       }
       return 2;
   },$a1,$a2);
   print_r($newarr); // 2 2 1
   ~~~

5. 关于foreach引用面试题

   ~~~php
   $arr = [1,2,3];
   foreach($arr as &$v){} //执行结束时$v是$arr[2]的引用
   foreach($arr as $v){}  //遍历是不断改变$arr[2]的值 开始是1 然后 2 在后 2
   print_r($arr); //1 2 2
   ~~~

6. 数组合并的运算 `+`  `array_merge` `array_merge_recursive` `array_combine` 区别

   ~~~php
   //+ 和 array_merge两种用法类似，都是完全合并两个数组，不同在于:
   //$a,$b 两个数组中有重复的key，+运算中以$a(前一个数组)为准，会覆盖掉后面的数组相同key的值。array_merge则相反，后面的会覆盖掉前面的。
   //array_merge_recursive 如果遇到相同的key则会递归创建一个二维数组如：
     ["b"]=>
     string(2) "bb"
     ["c"]=>
     array(2) {
       [0]=>
       string(2) "cc"
       [1]=>
       string(2) "cf"
     }
     ["A"]=>
     string(2) "aa"
   ~~~

7. 英文字母的ascII 

   A-Z 65-90  a-z 97-122
   
8. json_encode 有时会失败，多数是编码的问题，需要转换成utf-8才可以

   *返回一个整型（integer），这个值会是以下的常量之一：*

   | `JSON_ERROR_NONE`                  | 没有错误发生                                                 |           |
   | ---------------------------------- | ------------------------------------------------------------ | --------- |
   | `JSON_ERROR_DEPTH`                 | 到达了最大堆栈深度                                           |           |
   | `JSON_ERROR_STATE_MISMATCH`        | 无效或异常的 JSON                                            |           |
   | `JSON_ERROR_CTRL_CHAR`             | 控制字符错误，可能是编码不对                                 |           |
   | `JSON_ERROR_SYNTAX`                | 语法错误                                                     |           |
   | `JSON_ERROR_UTF8`                  | 异常的 UTF-8 字符，也许是因为不正确的编码。                  | PHP 5.3.3 |
   | `JSON_ERROR_RECURSION`             | One or more recursive references in the value to be encoded  | PHP 5.5.0 |
   | `JSON_ERROR_INF_OR_NAN`            | One or more [`NAN`](http://tw1.php.net/manual/zh/language.types.float.php#language.types.float.nan) or [`INF`](http://tw1.php.net/manual/zh/function.is-infinite.php) values in the value to be encoded | PHP 5.5.0 |
   | `JSON_ERROR_UNSUPPORTED_TYPE`      | 指定的类型，值无法编码。                                     | PHP 5.5.0 |
   | `JSON_ERROR_INVALID_PROPERTY_NAME` | 指定的属性名无法编码。                                       | PHP 7.0.0 |
   | `JSON_ERROR_UTF16`                 | 畸形的 UTF-16 字符，可能因为字符编码不正确。                 | PHP 7.0.0 |

   echo json_last_error();

   //这里也可以是json_decode

   //错误码对照

   0 JSON_ERROR_NONE

   1 JSON_ERROR_DEPTH

   2 JSON_ERROR_STATE_MISMATCH

   3 JSON_ERROR_CTRL_CHAR

   4 JSON_ERROR_SYNTAX

   5 JSON_ERROR_UTF8

   6 JSON_ERROR_RECURSION

   7 JSON_ERROR_INF_OR_NAN

   8 JSON_ERROR_UNSUPPORTED_TYPE

   ~~~php
   //可以用如下代码转码
   $data = serialize($dataarr);
   $encode = mb_detect_encoding($data, array("ASCII", 'UTF-8', 'GB2312', "GBK", 'BIG5', 'EUC-CN'));//判断编码v
   $string = iconv($encode, 'UTF-8', $matches[0][0]); //全部转utf-8
   ~~~

   