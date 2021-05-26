Windows版本

# 一、环境搭建

环境指开发语言、应用服务器、IDE 三个。

开发环境选择的是Java，应用服务器选择的是开源的Tomcat， IDE可以选择开源的eclipse，但是Intellij IDEA功能更强大。

## 1、安装Java

搜索引擎中搜索Java，安装JDK。

配置环境变量：我的电脑-属性-高级系统设置-环境变量

变量名输入JAVA_HOME，变量值选择之前安装的jdk目录。再找到PATH目录，双击进去，点击新建，选择jdk下的bin目录。

打开cmd命令行后，输入java和javac就会有变化了。可以输入

~~~
java -version
~~~

查看java版本号

## 2、安装Tomcat

搜索引擎中搜索Tomcat，这里选择免安装版。将压缩包中的内容解压到一个文件夹下，打开解压后的文件夹，bin

**点击startup.bat就能启动Tomcat服务**。此时在浏览器的URL栏中输入localhost:8080就能显示Tomcat的界面。

如果输入localhost:8080/docs就会跳转到docs目录下。

最后点击shutdown.bat关闭Tomcat服务

## 3、安装IDE

到Intellij IDEA官网上下载。

安装完成后，新建项目，在**Libraries and Frameworks中选择Web Profile**，next后修改Artifact的值。最终就能创建带有Web框架的项目了。or新建项目，右键选择Add Framework，同样选择Web Profile。

Run-edit configuration， 左上角添加Tomcat Server-local。可以自己命名Server的名称。在Application server选择自己的Tomcat路径。然后在Deployment的右边添加Artifact，此处可以修改Applcation context的值。

应用后就可以点击Run来启动Tomcat服务了。



# 三、web安全

WebGoat是一个故意不安全的J2EE Web应用程序，旨在教授Web应用程序安全性课程。 在每节课中，用户必须通过利用WebGoat应用程序中的实际漏洞来证明他们对安全问题的理解。 例如，用户必须使用SQL注入来窃取伪造的信用卡号。 该应用程序是一个现实的教学环境，可为用户提供提示和代码以进一步解释该课程。

## 1、安装WebGoat并使用

下载WebGoat，解压并双击webgoat.bat。

这里需要科学上网，到http://code.google.com/p/webgoat上进行下载5.4版本的webgoat。

双击webgoat.bat的结果如下

~~~cmd
五月 26, 2021 7:58:49 下午 org.apache.catalina.core.AprLifecycleListener init
信息: The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: E:\ZJU\Course\Secure Programming\Lab1\WebGoat-5.4\java\bin;C:\WINDOWS\Sun\Java\bin;C:\WINDOWS\system32;C:\WINDOWS;C:\Program Files\Common Files\Oracle\Java\javapath;C:\Program Files (x86)\Common Files\Oracle\Java\javapath;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;C:\Program Files (x86)\NVIDIA Corporation\PhysX\Common;C:\Program Files\NVIDIA Corporation\NVIDIA NvDLISR;C:\Program Files\MATLAB\R2020a\bin;C:\Program Files\Java\jdk-15.0.1\bin;C:\Program Files\Java\jdk-15.0.1\jre\bin;D:\Program Files (x86)\Git\cmd;D:\Program Files (x86)\x86_64-8.1.0-release-win32-sjlj-rt_v6-rev0\mingw64\bin;C:\Program Files (x86)\Windows Kits\8.1\Windows Performance Toolkit\;C:\Users\ASUS\AppData\Local\Programs\Python\Python38\Scripts\;C:\Users\ASUS\AppData\Local\Programs\Python\Python38\;C:\Users\ASUS\AppData\Local\Microsoft\WindowsApps;C:\Program Files\Bandizip\;D:\Program Files (x86)\Microsoft VS Code\bin;D:\Program Files\Fiddler;D:\Program Files\IntelliJ IDEA Educational Edition 2020.2.3\bin;;D:\Program Files\IntelliJ IDEA 2020.2.3\bin;;;D:\Program Files\JetBrains\PyCharm Community Edition 2020.3.3\bin;;.
五月 26, 2021 7:58:49 下午 org.apache.catalina.startup.SetAllPropertiesRule begin
警告: [SetAllPropertiesRule]{Server/Service/Connector} Setting property 'maxSpareThreads' to '75' did not find a matching property.
五月 26, 2021 7:58:49 下午 org.apache.coyote.AbstractProtocol init
信息: Initializing ProtocolHandler ["http-bio-127.0.0.1-80"]
五月 26, 2021 7:58:49 下午 org.apache.coyote.AbstractProtocol init
信息: Initializing ProtocolHandler ["ajp-bio-8009"]
五月 26, 2021 7:58:49 下午 org.apache.catalina.startup.Catalina load
信息: Initialization processed in 308 ms
五月 26, 2021 7:58:49 下午 org.apache.catalina.core.StandardService startInternal
信息: Starting service Catalina
五月 26, 2021 7:58:49 下午 org.apache.catalina.core.StandardEngine startInternal
信息: Starting Servlet Engine: Apache Tomcat/7.0.27
五月 26, 2021 7:58:49 下午 org.apache.catalina.startup.HostConfig deployWAR
信息: Deploying web application archive E:\ZJU\Course\Secure Programming\Lab1\WebGoat-5.4\tomcat\webapps\WebGoat.war
五月 26, 2021 7:58:50 下午 org.apache.catalina.startup.HostConfig deployDirectory
信息: Deploying web application directory E:\ZJU\Course\Secure Programming\Lab1\WebGoat-5.4\tomcat\webapps\host-manager
五月 26, 2021 7:58:50 下午 org.apache.catalina.startup.HostConfig deployDirectory
信息: Deploying web application directory E:\ZJU\Course\Secure Programming\Lab1\WebGoat-5.4\tomcat\webapps\manager
五月 26, 2021 7:58:50 下午 org.apache.catalina.startup.HostConfig deployDirectory
信息: Deploying web application directory E:\ZJU\Course\Secure Programming\Lab1\WebGoat-5.4\tomcat\webapps\ROOT
五月 26, 2021 7:58:50 下午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["http-bio-127.0.0.1-80"]
五月 26, 2021 7:58:50 下午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["ajp-bio-8009"]
五月 26, 2021 7:58:50 下午 org.apache.catalina.startup.Catalina start
信息: Server startup in 497 ms

~~~

最后显示Server startup表示服务器已经启动。此时就可以在浏览器的URL栏中输入http://localhost/webgoat/attack.**注意如果点击的是webgoat.bat，端口号默认为80**。


