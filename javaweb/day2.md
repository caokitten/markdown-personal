浏览器发送请求，到最终服务器调用Servlet中的方法的过程

- 获取url，浏览器发送request
- Tomcat服务器接受request，截取路径/crm/fdsa/fd/saf/d/sa/fd/sa/fd 
- Tomcat服务器根据路径找到crm项目，在xml文件中查找对应的Servlet：com.bjpowernode.servlet.HelloServlet
- 通过反射机制，创建com.bjpowernode.servlet.HelloServlet的对象，调用该对象的service方法

### 向浏览器响应一段html代码

```java
public void service(ServletRequest request,ServletResponse response) throws ServletException,IOException{
        //输出消息到控制台上
        response.setContentType("text/html");
        //设置响应类型，建议使用在获取流前设置
        response.setContentType("text/html");
        //将信息输出到浏览器上，需要调用ServletResponse接口
        //该输出流由Tomcat自动维护
        PrintWriter out = response.getWritter();
        //将信息输出到浏览器上
        out.print("Hello!");
        //响应一段html代码
        out.print("<h1>hello servlet!</h1>");
}
```

### 连接数据库

```java
public void service(ServletRequest request,ServletResponse response) throws ServletException,IOException{
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try{
        //注册驱动
    	Class.forName('com.mysql.cj.jdbc.Driver');
        //获取连接
        String url = "jdbc:mysql://locathost:3306/bjpowernode";
        String user = "root";
        String password = "1234";
        conn = DriverManager.getConnection(url,user,password);
        //获取预编译的数据库操作对象
        String sql = "select no,name from t_studet";
        ps = conn.prepareStatement(sql);
        //执行sql
        rs = ps.executeQuery();
        //处理查询结果集
        while(rs.next()){
            String no = rs.getString("no");
            String name = rs.getString("name");
        }
    }catch(Exception e){
        e.printStackTrace;
    }finally{
        //释放资源
        if(rs != null){
            try{
                rs.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
        if(ps != null){
            try{
                ps.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
        if(conn != null){
            try{
                conn.close();
            }catch(Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

### Servlet生命周期

1. Tomcat又称Web容器,存储并管理Servlet对象

2. 自己new的Servlet对象不会被Tomcat管理,因为不在Web容器中

3. Web容器底部有一个HashMap集合,存储请求路径(key)与Servlet对象(value)间的对应关系

4.  默认情况下服务器启动后至请求接收前不会创建对象,可在xml文件对应Servlet对象下配置

   ```xml
   <Servlet>
       <!--值越小优先级越高-->
   	<load-on-startup>0</load-on-startup>
   </Servlet>
   ```

5. 接收到请求后创建对象并调用init()方法

6. Servlet对象是单例的,即只实例化一次,但并非单例模式,因为构造方法非私有化

7. destroy()方法在服务器关闭Servlet对象销毁之前调用:

   无参构造-->init()-->service()-->destroy()-->销毁

