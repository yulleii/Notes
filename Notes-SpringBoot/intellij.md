# 更换Maven阿里源

打开IntelliJ IDEA->Settings ->Build, Execution, Deployment -> Build Tools > Maven

点击User Setting File 以及Local repository的`Override`

一般情况下在c:\\`UsersName`\xx.m2\这个目录下面没有settings.xml文件，我们可以新建一个settings.xml，复制以下内容到文件

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">

~~~xml
  <mirrors>
    <mirror>  
        <id>alimaven</id>  
        <name>aliyun maven</name>  
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
        <mirrorOf>central</mirrorOf>          
    </mirror>  
  </mirrors>
~~~

# 快捷方式

**try-catch**：ctrl+alt+t

