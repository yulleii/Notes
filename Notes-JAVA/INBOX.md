# 如何在Windows上配置Java环境变量

- 新建系统变量：变量名：*JAVA_HOME*；变量值：`D:\Java\jdk1.8.0_73`。这里的变量值是JDK的安装包位置。

- 新建系统变量：变量名：*CLASSPATH*；变量`.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;`

- 找到系统变量*Path*进行编辑，在该变量最后添加`%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;`

