![image-20250910111435379](./../../../../AppData/Roaming/Typora/typora-user-images/image-20250910111435379.png)

##### Web服务器软件

- Tomcat（底层由Java实现）
- jetty

##### 应用服务器软件

- jBOSS（内置了一个Tomcat服务器）
- WebLogic
- WebSphere

两者关系：

​	应用服务器包含于Web服务器，应用服务器实现了JavaEE的所有规范（13个），Web服务器只实现了JavaEE中的Serviet+JSP两个核心的规范

#### Tomcat

​	启动：bin目录下startup.bat文件（windows处理的dos命令，.sh文件为Linux处理的shell命令），实质最后执行catalina.bat文件，该文件配置项：MAINCLASS为main方法所在的类（Java由main方法为程序入口）

- 文件夹：
  - bin：命令文件目录
  - conf：配置文件目录
  - lib：核心文件目录
  - logs：日志文件目录
  - temp：临时文件目录
  - webapps：web应用文件目录
  - work：JSP文件翻译之后的Java文件目录与编译之后的class文件目录

### webapp

1. 页面中的超链接的ip地址与端口号是可以省略的

2. 一个路径对应一个servlet程序

3. 静态资源：写死在代码中的静态元素

   动态资源：由数据库传输的动态数据

4. 三种规范：

   1. Servlet规范：WebServer与Webapp解耦合
   2. HTTP协议：Browser与WebServer传输协议
   3. JDBC规范：WebServer与DBServer连接

### Servlet

1. 本质：request=>Servlet：根据请求调用程序=>response

2. crm

   - html
   - css
   - javascript
   - image
   - WEB_INF
     - classes：存放.class文件
     - lib：存放.jar文件（第三方包）
     - web.xml：配置文件
   - ......

3. java小程序必须实现Servlet接口

4. JavaEE8=>JakartaEE9

5. 对于该小程序，仅需将源码编译后的.class文件存放在classes文件中即可

6. 关于环境配置：

   - JAVA_HOME：jdk位置
   - CATALINA_HOME：Tomcat的根目录位置
   - PATH：bin目录位置
   - CLASSPATH：Servlet-api.jar文件位置

7. Servlet注册：

   ![image-20250911083904776](./../../../../AppData/Roaming/Typora/typora-user-images/image-20250911083904776.png)