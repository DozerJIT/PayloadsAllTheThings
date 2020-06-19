# PHP序列化与反序列化

## 摘要

* [什么是序列化与反序列化](#什么是序列化与反序列化)
* [PHP的序列化与反序列化](#PHP的序列化与反序列化)
* [实战案例](#实战案例1-简单序列化)
* [实在案例2](#实战案例2-序列化伪协议)

## 什么是序列化与反序列化

序列化与反序列化是面向对象语言特有的表示形式

### 序列化

将变量转换为可保存或传输的字符串的过程

### 反序列化

把字符串转化为原来的变量

## PHP的序列化与反序列化

PHP序列化、反序列化方法主要有四种

### 1.serialize/unserialize
这两个是序列化和反序列化PHP中数据的常用函数
#### 例1：

```
<?php
$s='student';
$s_serialize=serialize($s);
echo $s_serialize;
echo "\n";
$s_unserialize=unserialize($s_serialize);
echo $s_unserialize;
?>

//
输出结果：
s:7:"student";
student
```

序列化对于不同类型得到的字符串格式为：

```
String  -> s:size:value;
Integer -> i:value;
Boolean -> b:value;(保存1或0)
Null    -> N;
Array   -> a:size:{key definition;value definition;(repeated per element)}
Object  -> O:strlen(object name):object name:object size:{s:strlen(property name):property name:property definition;(repeated per property)}
```

#### 例2：

如果对象的类中定义了__sleep或__wakeup方法，则会在序列化时调用__sleep，反序列化时调用__wakeup

```
<?php
class A{
    private $var;
    public $var2="b";
    
    function __construct(){
        $this->var=mt_rand();
    }
    
    function __sleep(){
        return array("var","var2");
    }
    
    function __wakeup(){
        $this->var=mt_rand();
    }
    
    function __toString(){
        return "class[".__CLASS__."]:{var:$this->var,var2:$this->var2}\n";
    }
}
$a=new A();
echo $a;
$str=serialize($a);
echo "$str\n";
$b=unserialize($str);
echo $b;
?>

//
输出结果： 
class[A]:{var:1139586461,var2:b}
O:1:"A":2:{s:6:" A var";i:1139586461;s:4:"var2";s:1:"b";}
class[A]:{var:904643720,var2:b}
```
![Avatar](images/1.png)

此外，序列化对象的时候，只会保存属性值，不会保存常量的值。对于父类中的变量，则会保留。

### 2.json_encode/json_decode

例：
```
<?php
$a=array('a'=>'Apple','b'=>'boy','c'=>'cat');
//序列化数组
$s=json_encode($a);
echo $s;
//反序列化
$o=json_decode($s);
?>
//结果：{"a":"Apple","b":"boy","c":"cat"}
```

### 3.var_export/eval
var_export函数把变量作为一个字符串输出；eval把字符串当成PHP代码执行，反序列化得内容

例：
```
<?php
$a = array('a' => 'Apple' ,'b' => 'banana' , 'c' => 'Coconut');
//序列化数组
$s = var_export($a , true);
echo $s;
//输出结果： array ( 'a' => 'Apple', 'b' => 'banana', 'c' => 'Coconut', )
echo '<br /><br />';

//反序列化
eval('$my_var=' . $s . ';');

print_r($my_var);
//输出结果：Array ( [a] => Apple [b] => banana [c] => Coconut )
?>
```

### 4.wddx_serialize_value/wddx deserialize

wddx_serialize_value函数可以序列化数组变量，并以XML字符串形式输出

例：
```
<?php
$a = array('a' => 'Apple' ,'b' => 'banana' , 'c' => 'Coconut');
//序列化数组
$s = wddx_serialize_value($a);
echo $s;
//输出结果（查看输出字符串的源码）：<wddxPacket version='1.0'><header/><data><struct><var name='a'><string>Apple</string></var><var name='b'><string>banana</string></var><var name='c'><string>Coconut</string></var></struct></data></wddxPacket>
echo '<br /><br />';
//反序列化
$o = wddx_deserialize($s);
print_r($o);
//输出结果：Array ( [a] => Apple [b] => banana 1 => Coconut )
?>
```

## 实战案例1-简单序列化
![Avatar](images/2.png)

__construct构造函数和__destruct析构函数的魔术方法是默认执行的，一个初始化对象，一个在对象生命周期结束时自动执行析构

```
要点：  1.preg_match(’/[oc]:\d+:/i’, $var)的绕过
       2.unserialize时__wakeup的绕过
```
**1.正则匹配在对象长度之前添加一个'+'号可以绕过
2.__wakeup绕过（CVE漏洞）：成员属性数目大于实际数目时可绕过wakeup方法(CVE-2016-7124)**

![Avatar](images/3.png)

根据提示我们需要访问的file是fl4g.php，通过var在url传参绕过__wakeup（不然只能访问index.php)

![Avatar](images/31.png)

还需要base64编码，最后得到flag

## 实战案例2-序列化+伪协议
![Avatar](images/4.png)
![Avatar](images/5.png)

如图，提示不是admin，且查看隐藏源码提醒有file_get_contents，file_get_contents() 函数把整个文件读入一个字符串中，这里的字符串是$user参数接受的，利用它的文件封装协议来读取$file参数include的文件。
既然如此，我们将指定字符串作为文件传进去，然后再将user的内容读出来

php伪协议绕过payload:

![Avatar](images/6.png)

绕过成功返回hello admin!Secret is in class.php
只绕过$user是远远不够的，我们需要读取class.php源码，使用php伪协议读取源码

![Avatar](images/7.png)

解码：

![Avatar](images/8.png)

顺便读取index1.php源码：

![Avatar](images/9.png)

注意到flag应该就在f1a9.php里，这里我们无法直接读取f1a9.php，因为有正则匹配，匹配到f1a9就直接执行exit()函数。
查看class.php，得出结论：当Read方法当做字符串执行时，会自动执行 __tostring 方法，方法中写了如果file文件存在，那么就输出file文件中的内容。所以我们要构造一个Read类型的参数，并把这个参数让pass传进去,且这个file的值要是class.php。这里接着用序列化里面插入php伪协议绕过，或者直接序列化f1a9也是可行的。

![Avatar](images/10.png)
![Avatar](images/11.png)

最后得到flag
我在服务器上搭建了题目环境：[www.youkilearning.top:8099/index1.php](http://www.youkilearning.top:8099/index1.php)

## Reference
* [深度剖析额PHP序列化和反序列化](https://www.cnblogs.com/youyoui/p/8610068.html)
* [PHP序列化与反序列化](https://www.cnblogs.com/dayin1/p/11465832.html)
* [PHP题目复现](https://www.cnblogs.com/Pinging/p/8278168.html)