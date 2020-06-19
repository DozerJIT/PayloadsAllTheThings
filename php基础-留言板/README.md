# PHP基础

- [php基本格式](#php基本格式)
- [php注释](#php注释)
- [php变量](#php变量)
- [php常量](#php常量)
- [输入&输出](#输入&输出)
- [php数据类型](#php数据类型)
- [类型比较](#类型比较)
- [运算符](#运算符)
- [流程控制语句](#流程控制语句)
- [超级全局变量](#超级全局变量)
- [函数](#函数)
- [变量作用域](#变量作用域)
- [留言板](#留言板)



### php基本格式

- <?php开头 ?>结尾    推荐格式

- <u><? ?>短标签格式使用：php.ini文件中修改short_open_tag 重启apache</u>

- php中每个代码行必须以 ' ; ' 结束<u>（ 最后一句代码也可以不加;）</u>

- <u>php文件名&路径不可以含有中文</u>

- php忽略语句之间空白（空格，制表符，换行符）

  ```
  <?php
  echo "hello world";			//输出 hello world
  ?>
  ```

  

### php注释

- //  单行注释  或者  # 单行注释 
- /* 多行注释*/

### php变量

- php没有申明变量的命令,php会根据变量的值，自动把变量转换为正确的数据类型，所以是弱类型语言。在强类型的编程语言中，我们必须在使用变量前先声明（定义）变量的类型和名称。

- 变量以$开始，类似 $a=123;

- <u>php变量区分大小写。关键字，函数，自定义的类与函数不区分大小写</u>

- <u>变量名称必须以字母或下划线开头</u>

  ```
  <?php
  $x=10;
  echo $x;			// 定义x变量值为10 并输出x
  ?>
  ```

### php常量

- 常量是一个简单的标识符，值在脚本中不可以改变

- 使用define()函数:define(名称，值，默认大小写敏感，设定ture为不敏感)

  ```
  define("EFG", "ABCD");
  echo EFG;
  输出			// 输出 ABCD
  ```

### 输入&输出

- echo    print   都可以加或不加括号     

  - echo更快	echo()没有返回值	echo可以输出一个或多个字符串
  - print()有返回值（值为1）      echo只能输出一个字符串

- var_dump() 返回变量的 数据类型 和 值

  ```
  <?php
  $a='hello world';
  var_dump($a);      //输出 string(11) "hello world"
  ?>
  ```

  

### php数据类型

- String(字符串)，integer(整型)，Float（浮点型），Boolean（布尔型），Array（数组），Object（对象），Null（空值）
- 你可以详细的使用以下文档来了解
- [php数据类型](https://www.runoob.com/php/php-datatypes.html)
- [php数据类型](http://c.biancheng.net/view/6113.html)
- [官方文档](https://www.php.net/manual/zh/language.types.php)

### 类型比较

- ==          只比较值，不比较类型
- ===        先比较类型再比较值
- 你可以详细的使用以下文档来了解
- [php类型比较](https://www.runoob.com/php/php-types-comparisons.html)
- [官方文档](https://www.php.net/manual/zh/language.operators.comparison.php)

### 运算符

- 值得一提的是 
  - <=>太空船运算符  a<=>b a>b值为1 a<b值为-1 a=b就是0
  - <> 不等于 
  - 三元运算符 A?B:C   A ture->返回B false返回C
  - 并置运算符就是用来连接的" . "符号
- [附上链接](https://www.runoob.com/php/php-operators.html)

### 流程控制语句

- 除了大家常见的if，switch之类的语句，php里面还有一个foreach

  ```
  <?php
  
  $t=date("H");
  if ($t<"20")
  {
      echo "Have a good day!";
  }
  
  
  
  $t=date("H");
  if ($t<"20")
  {
      echo "Have a good day!";
  }
  else
  {
      echo "Have a good night!";
  }
  
  $t=date("H");
  if ($t<"10")
  {
      echo "Have a good morning!";
  }
  else if ($t<"20")
  {
      echo "Have a good day!";
  }
  else
  {
      echo "Have a good night!";
  }
  
  $favcolor="red";
  switch ($favcolor)
  {
      case "red":
      echo "Your favorite color is red!";
      break;
      case "blue":
      echo "Your favorite color is blue!";
      break;
      case "green":
      echo "Your favorite color is green!";
      break;
      default:
      echo "Your favorite color is neither red, blue, or green!";
  }
  
  //循环
  $i=1;
  while($i<=5)
  {
      echo "The number is " . $i . "<br>";
      $i++;
  }
  
  
  $i=1;
  do
  {
      $i++;
      echo "The number is " . $i . "<br>";
  }
  while ($i<=5);
  
  for ($i=1; $i<=5; $i++)
  {
      echo "The number is " . $i . "<br>";
  }
  
  
  $x=array("A","B","C");
  foreach ($x as $value)
  {
      echo $value;          //输出  ABC
  }
  
  ?>
  ```

- [附上链接](https://www.php.net/manual/zh/language.control-structures.php)

### 超级全局变量

- 其实除了GLOBALS，其他的也是为了接受php脚本以外的变量。如接受GET POST的值，接收上传的文件等
- php有预定义的超级全局变量，你可以在一个脚本的全部作用域使用，在往后的学习中你会逐渐遇到，这里就不一一解释了
- $GLOBALS
- $_SERVER  
- $_REQUEST 
- $_POST 
- $_GET                  
- $_FILES                
- $_ENV
- $_COOKIE
- $_SESSION
- [附上文档](https://www.php.cn/php-weizijiaocheng-415135.html)

### 函数

- php有很多内置函数，你也同样可以自定义

- [函数](https://www.runoob.com/php/php-ref-array.html)

- 函数名称以字母或下划线开头

  ```
  <?php
  function hello()
  {
      echo "hello world";
  }
  hello();               //输出hello world
  ?>
  ```

- 传入参数

  ```
  <?php
  function hello($name)
  {
      echo "hello ".$name;
  }
  $a="world";
  hello($a);              //输出hello world 
  ?>
  ```

- 返回值 return

  ```
  <?php
  function hello()
  {
      return "hello world";
  }
  $a=hello();   
  echo $a;				//输出hello world
  echo hello();			//输出hello world
  ?>
          
  ```

  

### 变量作用域

- local（局部）, global（全局）, static（静态）, parameter（参数）

- 局部&全局变量：

  1. 函数内部申明的变量是局部变量，如果想在函数内部调用全局则需要加上global关键字。

     ```
     <?php
     $a = 1;
     function Sum()
     {	$a = 0;
         global $a;
         echo $a;
     }
     echo $a;                   // 输出 1
     ?>
     
     ```

     

  2. php将所有的全局变量存储在一个名叫$GLOBALS[]的数组中，可以在函数内部调用

     ```
     <?php
     $x=10;					
     echo $GLOBALS['x']."</br>";	 //输出 10
     function hello()
     {
         $x=5;
         echo $GLOBALS['x']."</br>";  //输出 10
     }
     hello();
     ?>
     ```

     

- Static：不会被销毁（通常函数结束，该局部变量会销毁），下次还可以用的，局部变量，使用static关键词

- 参数作用域：通过调用代码将值传递给函数的局部变量

### 参考

- https://www.php.net/manual/zh/
- <https://www.runoob.com/php/php-tutorial.html>
- <https://www.runoob.com/w3cnote/php-basic-summary.html>
- <http://c.biancheng.net/php/>

# 留言板

### 数据库的配置 config.php

```
<?php
//date_default_timezone_set('Asia/Shanghai');//设置时区
header('Content-type:text/html;charset=utf-8');//设置默认字符

//定义数据库连接参数
define('DBHOST', '127.0.0.1');//将localhost修改为数据库服务器的地址
define('DBUSER', 'root');	  //连接mysql的用户名
define('DBPW', 'root');       //连接mysql的密码
define('DBNAME', 'mine');     //数据库名
define('DBPORT', '3306');     //默认mysql端口3306

?>

```

### 数据库的初始化 install.php

```
<?php

include_once 'config.php';

$dbhost=DBHOST;
$dbuser=DBUSER;
$dbpw=DBPW;
$dbname=DBNAME;

if(isset($_POST['submit'])){
    //判断数据库能否连接
    if(!@mysqli_connect($dbhost, $dbuser, $dbpw)){
        exit('数据连接失败，请仔细检查config.php参数的配置');
    }
    $link=mysqli_connect(DBHOST, DBUSER, DBPW);
    
    //检查，如果database名已存在,则直接删除（嘿嘿）
    $drop_db="drop database if exists $dbname";
    if(!@mysqli_query($link, $drop_db)){
        exit('Error，请仔细检查当前用户是否有操作权限');
    }
 
    //创建数据库
    $create_db="CREATE DATABASE $dbname";
    if(!@mysqli_query($link,$create_db)){
        exit('数据库创建失败，请仔细检查当前用户是否有操作权限');
    }

    //选择数据库
    if(!@mysqli_select_db($link, $dbname)){
        exit('数据库选择失败，请仔细检查当前用户是否有操作权限');
    }

    //创建表message123
    $create_message=
        "CREATE TABLE IF NOT EXISTS `message123` (`id` int(10) unsigned NOT NULL AUTO_INCREMENT,`content` varchar(255) NOT NULL,`time` datetime NOT NULL,
     PRIMARY KEY (`id`))ENGINE=MyISAM  DEFAULT CHARSET=utf8 AUTO_INCREMENT=56";
    if(!@mysqli_query($link,$create_message)){
        exit('创建message表失败，请仔细检查当前用户是否有操作权限');
    }
    echo "数据库初始化完成！！";
}
?>

<!DOCTYPE html>
<html>
<head>
	<title>Install</title>
</head>
<body>

<form method="post">
    <input type="submit" name="submit" value="安装/初始化"/>
</form>
</body>
</html>

```

### 自定义函数库 function.php

```
<?php
//db connect
function connect($host=DBHOST,$username=DBUSER,$password=DBPW,$databasename=DBNAME,$port=DBPORT){
	$link=@mysqli_connect($host, $username, $password, $databasename, $port);
	if(mysqli_connect_errno()){
// 		exit(mysqli_connect_error());
	    exit('数据库连接失败，请检查config.php配置文件');
	}
	mysqli_set_charset($link,'utf8');
	return $link;
}

//这个函数其实也没必要，留着也行，你也可以直接用
function execute($link,$query){
	$result=mysqli_query($link,$query);
	return $result;
}

//sql预处理，防SQL注入
function escape_exe_s($link,$query,$A)
{
	$stmt=$link->prepare($query);
	$stmt->bind_param('s',$A);
	$stmt->execute();

}

//同上
function escape_exe_i($link,$query,$A)
{
	$stmt=$link->prepare($query);
	$stmt->bind_param('i',$A);
	$stmt->execute();
		//删除后刷新message
		if(mysqli_affected_rows($link)==1){
        	echo "<script type='text/javascript'>document.location.href='lys.php'</script>";}
    	else{
        	$html.="<p id='op_notice'>删除失败,请重试并检查数据库是否还好!</p>";}
}


?>
```

### 留言板 lys.php

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>我不是一个留言板</title>
</head>
<body>
<form method="post">

<textarea  name="message"></textarea><br />
<input  type="submit" name="submit" value="留言" />

</form>

<?php

include_once 'function.php';
include_once 'config.php';
error_reporting(0);
$link=connect();



//插入message
//array_key_exists()
if(array_key_exists("message",$_POST) && $_POST['message']!=null)
{ 
     //sql预处理语句 
     $query="insert into message123(content,time) values(?,now())";
     //防注入函数位于function.php
     escape_exe_s($link,$query,$_POST['message']);
}




//查询message
$query="select * from message123";
$result=execute($link, $query);
//mysqli_fetch_assoc() 
while($data=mysqli_fetch_assoc($result))
{
    //htmlspecialchars()转义预定义字符它们是< > " ' &，其实很危险该函数可以被绕过
    //这里也是全篇唯一的XSS处理，但我觉得应该够了
    $content=htmlspecialchars( $data['content'] );
    echo "<p class='con'>{$content}</p><a href='?id={$data['id']}'>删除</a>";
}    




//删除message
//sql预处理语句
$querydel="delete from message123 where id=?";
//防注入函数位于function.php
escape_exe_i($link,$querydel,$_GET['id']);

?>

</body>
</html>
```

###### [这里最后附上一个已经写好的](http://106.12.28.183/lys/lys.php)
