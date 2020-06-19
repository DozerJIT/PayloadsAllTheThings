# PHP 弱类型和魔术hashes

PHP提供了两种方法来比较两个数值：

- 宽松的比较使用的是`== 或者 !=` ：两个数值拥有相同的数值
- 严格比较使用的是`=== 或者 !==：`两个数值有相同的数据类型和相同的数值

## Type Juggling

### True statements

bool类型

```php
var_dump('0010e2'   == '1e3');           # true
var_dump('0xABCdef' == ' 0xABCdef');     # true PHP 5.0 / false PHP 7.0
var_dump('0xABCdef' == '     0xABCdef'); # true PHP 5.0 / false PHP 7.0
var_dump('0x01'     == 1)                # true PHP 5.0 / false PHP 7.0
var_dump('0x1234Ab' == '1193131');
```

```php
'123'  == 123
'123a' == 123
'abc'  == 0
```

```php
'' == 0 == false == NULL
'' == 0       # true
0  == false   # true
false == NULL # true
NULL == ''    # true
```

### NULL statements

NULL类型

```php
var_dump(sha1([])); # NULL
var_dump(md5([]));  # NULL
```

## Magic Hashes - Exploit

如果一个hash以“0e”开始(或者“0...0e”)后面跟的全是数字，PHP将会把这个hash当成一个浮点数

| Hash    | “Magic” Number / String |                          Magic Hash                          |                                                     Found By |
| ------- | ----------------------- | :----------------------------------------------------------: | -----------------------------------------------------------: |
| MD5     | 240610708               |               0e462097431906509019562988736854               | [@spazef0rze](https://twitter.com/spazef0rze/status/439352552443084800) |
| SHA1    | 10932435112             |           0e07766915004133176347055865026311692244           | Independently found by Michael A. Cleverly & Michele Spagnuolo & Rogdham |
| SHA-224 | 10885164793773          |   0e281250946775200129471613219196999537878926740638594636   | [@TihanyiNorbert](https://twitter.com/TihanyiNorbert/status/1138075224010833921) |
| SHA-256 | 34250003024812          | 0e46289032038065916139621039085883773413820991920706299695051332 | [@TihanyiNorbert](https://twitter.com/TihanyiNorbert/status/1148586399207178241) |
| SHA-256 | TyNOQHUS                | 0e66298694359207596086558843543959518835691168370379069085300385 |                                                              |

[@Chick3nman512](https://twitter.com/Chick3nman512/status/1150137800324526083)

```php
<?php
var_dump(md5('240610708') == md5('QNKCDZO')); # bool(true)
var_dump(md5('aabg7XSs')  == md5('aabC9RqS'));
var_dump(sha1('aaroZmOk') == sha1('aaK1STfY'));
var_dump(sha1('aaO8zKZF') == sha1('aa3OFF9m'));
?>
```

## References

* [Writing Exploits For Exotic Bug Classes: PHP Type Juggling By Tyler Borland](http://turbochaos.blogspot.com/2013/08/exploiting-exotic-bugs-php-type-juggling.html)
* [Magic Hashes - WhieHatSec](https://www.whitehatsec.com/blog/magic-hashes/)