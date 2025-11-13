### 关于路径中的项目名

- 前端路径中要加项目名
- 后端路径可不加项目名

### 一个纯 Servlet 实现的 crud 功能的 Webapp:

- 结构:

  - 欢迎页面: index.html

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Hello</title>
    </head>
    <body>
        <h1>Hello Web!</h1>
        <a href="/test/dept/list">GO To Form!</a>
    </body>
    </html>
    ```

  - 表单页面: DeptListServlet

    ```java
    package com.bigpowernode.javaweb.servlet;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    import java.io.PrintWriter;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    public class DeptListServlet extends HttpServlet {
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
            String ContextPath = request.getContextPath();
    
            response.setContentType("text/html");
            PrintWriter out = response.getWriter();
    
            out.print("           <!DOCTYPE html>");
            out.print("   <html lang='en'>");
            out.print("   <head>");
            out.print("       <meta charset='UTF-8'>");
            out.print("       <title>Title</title>");
            out.print("   </head>");
            out.print("   <body>");
            out.print("     <script type='text/javascript'>function del(dno){var res = window.confirm('确定删除?');if(res){document.location.href='"+ContextPath+"/dept/delete?deptno='+dno}}</script>");
            out.print("       <table border='1px' align='center' width='50%'>");
            out.print("           <tr>");
            out.print("               <th>序号</th>");
            out.print("               <th>部门编号</th>");
            out.print("               <th>部门名称</th>");
            out.print("               <th>操作</th>");
            out.print("           </tr>");
    
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
                conn = DBUtil.getConnection();
                //创建预编译语句对象
                pstmt = conn.prepareStatement("select deptno,dname from dept");
                //执行并获取结果集
                rs = pstmt.executeQuery();
    
                int id = 0;
                while (rs.next()) {
                    String deptno = rs.getString("deptno");
                    String dname = rs.getString("dname");
    
                    out.print("           <tr>");
                    out.print("               <th>"+(++id)+"</th>");
                    out.print("               <th>"+deptno+"</th>");
                    out.print("               <th>"+dname+"</th>");
                    out.print("               <th>");
                    out.print("                   <a href='javascript:void(0)' onclick='del("+deptno+")'>删除</a>");
                    out.print("                   <a href='"+ContextPath+"/dept/edit?deptno="+deptno+"'>修改</a>");
                    out.print("                   <a href='"+ContextPath+"/dept/detail?deptno="+deptno+"'>详细</a>");
                    out.print("               </th>");
                    out.print("           </tr>");
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }finally {
                DBUtil.close(conn, pstmt, rs);
            }
    
            out.print("       </table>");
            out.print("       <hr>");
            out.print("       <a href='"+ContextPath+"/add.html'>增加</a>");
            out.print("   </body>");
            out.print("   </html>");
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            doGet(req, resp);
        }
    }
    
    ```

  - 增加页面: add.html

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Add</title>
    </head>
    <body>
        <h1>新增部门</h1>
        <hr>
        <form action="/test/dept/save" method="post">
            部门编号<input type="text" name="deptno">
            <br>
            部门名称<input type="text" name="dname">
            <br>
            位置<input type="text" name="loc">
            <br>
            <input type="submit" name="提交">
        </form>
    </body>
    </html>
    ```

  - 增加实现: DeptSaveServlet

    ```java
    package com.bigpowernode.javaweb.servlet;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    
    public class DeptSaveServlet extends HttpServlet {
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
            String deptno = request.getParameter("deptno");
            String dname = request.getParameter("dname");
            String loc = request.getParameter("loc");
    
            int count = 0;
    
            Connection conn = null;
            PreparedStatement pstmt = null;
    
            try {
                conn = DBUtil.getConnection();
                pstmt = conn.prepareStatement("insert into dept(deptno, dname, loc) values(?,?,?)");
                pstmt.setString(1, deptno);
                pstmt.setString(2, dname);
                pstmt.setString(3, loc);
                count = pstmt.executeUpdate();
            } catch (SQLException e) {
                e.printStackTrace();
            }finally {
                DBUtil.close(conn,pstmt,null);
            }
    
            if (count > 0) {
                request.getRequestDispatcher("/dept/list").forward(request, response);
            }else
                request.getRequestDispatcher("/error.html").forward(request, response);
        }
    }
    
    ```

  - 删除实现: DeptDeleteServlet

    ```java
    package com.bigpowernode.javaweb.servlet;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    public class DeptDeleteServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
            String deptno = request.getParameter("deptno");
            int count = 0;
    
            Connection conn = null;
            PreparedStatement pstmt = null;
    
            try {
                conn=DBUtil.getConnection();
                //开启事务
                conn.setAutoCommit(false);
                pstmt=conn.prepareStatement("delete from dept where deptno=?");
                pstmt.setString(1, deptno);
                //返回值为影响了数据库中的记录量
                count = pstmt.executeUpdate();
                //事务提交
                conn.commit();
            } catch (SQLException e) {
                //回滚
                if(conn==null){
                    try {
                        conn.rollback();
                    } catch (SQLException ex) {
                        throw new RuntimeException(ex);
                    }
                }
                e.printStackTrace();
            }finally {
                DBUtil.close(conn,pstmt,null);
            }
    
            if(count>0){
                request.getRequestDispatcher("/dept/list").forward(request, response);
            }else
                request.getRequestDispatcher("/error.html").forward(request, response);
        }
    }
    
    ```

  - 修改页面: DeptEditServlet

    ```java
    package com.bigpowernode.javaweb.servlet;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    import java.io.PrintWriter;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    public class DeptEditServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
            response.setContentType("text/html");
            PrintWriter out = response.getWriter();
    
            String ContextPath = request.getContextPath();
    
            out.print("         <!DOCTYPE html>");
            out.print(" <html lang='en'>");
            out.print(" <head>");
            out.print("     <meta charset='UTF-8'>");
            out.print("     <title>Edit</title>");
            out.print(" </head>");
            out.print(" <body>");
            out.print(" <h1>修改部门</h1>");
            out.print(" <hr>");
            out.print(" <form action='"+ContextPath+"/dept/modify' method='post'>");
    
            String deptno = request.getParameter("deptno");
    
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
                conn = DBUtil.getConnection();
                pstmt = conn.prepareStatement("select dname,loc from dept where deptno = ?");
                pstmt.setString(1, deptno);
                rs = pstmt.executeQuery();
                if (rs.next()) {
                    String dname = rs.getString("dname");
                    String loc = rs.getString("loc");
    
                    out.print("                 部门编号<input type='text' name='deptno' value='"+deptno+"' readonly>");
                    out.print("     <br>");
                    out.print("                 部门名称<input type='text' name='dname' value='"+dname+"'>");
                    out.print("     <br>");
                    out.print("                 位置<input type='text' name='loc' value='"+loc+"'>");
                    out.print("     <br>");
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }finally {
                DBUtil.close(conn, pstmt, rs);
            }
    
            out.print("     <input type='submit' name='提交'>");
            out.print(" </body>");
            out.print(" </html>");
        }
    }
    
    ```

  - 修改实现: DeptModifyServlet

    ```java
    package com.bigpowernode.javaweb.servlet;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    import java.io.IOException;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.SQLException;
    
    public class DeptModifyServlet extends HttpServlet {
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
            String deptno = request.getParameter("deptno");
            String dname = request.getParameter("dname");
            String loc = request.getParameter("loc");
    
            int count = 0;
    
            Connection conn = null;
            PreparedStatement pstmt = null;
    
            try {
                conn = DBUtil.getConnection();
                pstmt = conn.prepareStatement("update dept set dname=?, loc=? where deptno=?");
                pstmt.setString(1, dname);
                pstmt.setString(2, loc);
                pstmt.setString(3, deptno);
                count = pstmt.executeUpdate();
            } catch (SQLException e) {
                e.printStackTrace();
            }finally {
                DBUtil.close(conn,pstmt,null);
            }
    
            if (count > 0) {
                request.getRequestDispatcher("/dept/list").forward(request, response);
            }else
                request.getRequestDispatcher("/error.html").forward(request, response);
        }
    }
    
    ```

  - 查询页面 DeptDetailServlet

    ```java
    package com.bigpowernode.javaweb.servlet;
    
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServlet;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    
    
    import java.io.IOException;
    import java.io.PrintWriter;
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
        public class DeptDetailServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
            response.setContentType("text/html");
            PrintWriter out = response.getWriter();
    
            out.print("         <!DOCTYPE html>");
            out.print(" <html lang='en'>");
            out.print(" <head>");
            out.print("     <meta charset='UTF-8'>");
            out.print("     <title>Detail</title>");
            out.print(" </head>");
            out.print(" <body>");
            out.print("  <h1>部门详情</h1><hr>");
    
            String deptno = request.getParameter("deptno");
    
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
    
                conn = DBUtil.getConnection();
                pstmt = conn.prepareStatement("select dname,loc from dept where deptno = ?");
                //替换字符
                pstmt.setString(1, deptno);
                rs = pstmt.executeQuery();
    
                if (rs.next()) {
                    String dname = rs.getString("dname");
                    String loc = rs.getString("loc");
    
                    out.print("                 部门编号: "+deptno+"<br>");
                    out.print("                 部门名称: "+dname+"<br>");
                    out.print("                 部门位置: "+loc+"<hr>");
                }
    
            } catch (SQLException e) {
                e.printStackTrace();
            }finally {
                DBUtil.close(conn, pstmt, rs);
            }
    
            out.print(" <input type='button' value='back' onclick='window.history.back()' />");
            out.print(" </body>");
            out.print(" </html>");
        }
    }
    
    ```

  - 错误页面: error.html

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>error</title>
    </head>
    <body>
    <h1>操作失败,<a href="javascript:void(0)" onclick="window.history.back()">返回</a></h1>
    </body>
    </html>
    ```

  - 数据库连接封装类: DBUtil

    ```java
    package com.bigpowernode.javaweb.servlet;
    
    import java.sql.*;
    import java.util.ResourceBundle;
    
    public class DBUtil {
    
        private static ResourceBundle resourceBundle = ResourceBundle.getBundle("resources.JDBC");
        private static String driver = resourceBundle.getString("driver");
        private static String url = resourceBundle.getString("url");
        private static String user = resourceBundle.getString("user");
        private static String password = resourceBundle.getString("password");
    
        static {
            try {
                Class.forName(driver);
            } catch (ClassNotFoundException e) {
                throw new RuntimeException(e);
            }
        }
    
        /**
         * 获取数据库连接对象
         * @return conn连接对象
         * @throws SQLException
         */
        public static Connection getConnection() throws SQLException {
            Connection conn = DriverManager.getConnection(url,user,password);
            return conn;
        }
    
        public static void close(Connection conn, Statement stmt, ResultSet rs) {
            if (rs != null) {
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (stmt != null) {
                try {
                    stmt.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    ```

  - 配置信息: web.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
             version="4.0">
    <!--Servlet配置-->
        <servlet>
            <servlet-name>list</servlet-name>
            <servlet-class>com.bigpowernode.javaweb.servlet.DeptListServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>list</servlet-name>
            <url-pattern>/dept/list</url-pattern>
        </servlet-mapping>
    
        <servlet>
            <servlet-name>detail</servlet-name>
            <servlet-class>com.bigpowernode.javaweb.servlet.DeptDetailServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>detail</servlet-name>
            <url-pattern>/dept/detail</url-pattern>
        </servlet-mapping>
    
        <servlet>
            <servlet-name>delete</servlet-name>
            <servlet-class>com.bigpowernode.javaweb.servlet.DeptDeleteServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>delete</servlet-name>
            <url-pattern>/dept/delete</url-pattern>
        </servlet-mapping>
    
        <servlet>
            <servlet-name>save</servlet-name>
            <servlet-class>com.bigpowernode.javaweb.servlet.DeptSaveServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>save</servlet-name>
            <url-pattern>/dept/save</url-pattern>
        </servlet-mapping>
    
        <servlet>
            <servlet-name>edit</servlet-name>
            <servlet-class>com.bigpowernode.javaweb.servlet.DeptEditServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>edit</servlet-name>
            <url-pattern>/dept/edit</url-pattern>
        </servlet-mapping>
    
        <servlet>
            <servlet-name>modify</servlet-name>
            <servlet-class>com.bigpowernode.javaweb.servlet.DeptModifyServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>modify</servlet-name>
            <url-pattern>/dept/modify</url-pattern>
        </servlet-mapping>
    </web-app>
    ```


### 转发与重定向

- 转发

  - 将该 Servlet 的 request 转发给其他 Servlet(一次请求)

    ```java
    request.getRequestDispatcher("目标路径").forward(request, response);
    ```

- 重定向

  - Servlet 向浏览器发送目标 Servlet 的路径, 浏览器自发地向服务器发送一次全新的指向该路径的请求(两次请求)

    ```java
    reponse.sendRedirect("目标路径");
    ```

  - 重定向中的路径由浏览器使用, 因此需要加项目名
  
  - 重定向第二次请求为 get 请求, 需要注意安全问题
  
### Servlet 注解 

- Servlet 不必配置在 web.xml 中, 而是直接写入到 java 类中

  - ```java
    jakarta.servlet.annotation.WebServlet
    ```

  - 在 Servlet 类上使用:

    ```java
    @WebServlet(
    
       	//Servlet名
        name = "UserServlet",
        //url模式
        urlPatterns = {"/users", "/user/*"},
        //服务器启动时创建并设置优先级为1
        loadOnStartup = 1,
        //初始化参数
        initParams = {
            @WebInitParam(name = "maxResults", value = "50"),
            @WebInitParam(name = "cachingEnabled", value = "true")
        }
        //...
    )
    ```

  - 当注解的属性名是 value 的时候，使用注解的时候，value 属性名是可以省略的, 当 value 只有一个时, 花括号是可以省略的:

    ```java
    @WebServlet("/dept/*")
    ```

### 利用重定向, Servlet 注解与模板设计模式优化 test 项目

- 关于模板设计:

  为避免类爆炸问题, 一个业务的实现对应一个类, 一个请求对应一个方法

  - 创建 DeptServlet 类并继承 HttpServlet

  - 重写 service 方法

  - 获取 Servlet Path 并匹配至对应方法:

  - ```java
    @WebServlet("/dept/*")
    public class DeptServlet extends HttpServlet {
    
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //获取Servlet Path
            String servletPath = request.getRequestURI().substring(request.getContextPath().length());
    
            if(servletPath.equals("/dept/list")){
                doList(request,response);
            }else if(servletPath.equals("/dept/save")){
                doSave(request,response);
            }else if(servletPath.equals("/dept/edit")){
                doEdit(request,response);
            }else if(servletPath.equals("/dept/delete")){
                doDel(request,response);
            }else if(servletPath.equals("/dept/modify")){
                doModify(request,response);
            }else if(servletPath.equals("/dept/detail")){
                doDetail(request,response);
            }
        }
    }
    ```

- 该业务的 java 代码的完整实现:
  
  ```java
  package com.bigpowernode.javaweb.servlet;
  
  import jakarta.servlet.ServletException;
  import jakarta.servlet.annotation.WebServlet;
  import jakarta.servlet.http.HttpServlet;
  import jakarta.servlet.http.HttpServletRequest;
  import jakarta.servlet.http.HttpServletResponse;
  
  import java.io.IOException;
  import java.io.PrintWriter;
  import java.sql.Connection;
  import java.sql.PreparedStatement;
  import java.sql.ResultSet;
  import java.sql.SQLException;
  
  @WebServlet("/dept/*")
  public class DeptServlet extends HttpServlet {
  
      @Override
      protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          //获取Servlet Path
          String servletPath = request.getRequestURI().substring(request.getContextPath().length());
  
          if(servletPath.equals("/dept/list")){
              doList(request,response);
          }else if(servletPath.equals("/dept/save")){
              doSave(request,response);
          }else if(servletPath.equals("/dept/edit")){
              doEdit(request,response);
          }else if(servletPath.equals("/dept/delete")){
              doDel(request,response);
          }else if(servletPath.equals("/dept/modify")){
              doModify(request,response);
          }else if(servletPath.equals("/dept/detail")){
              doDetail(request,response);
          }
      }
  
      private void doList(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  
          String ContextPath = request.getContextPath();
  
          response.setContentType("text/html");
          PrintWriter out = response.getWriter();
  
          out.print("           <!DOCTYPE html>");
          out.print("   <html lang='en'>");
          out.print("   <head>");
          out.print("       <meta charset='UTF-8'>");
          out.print("       <title>Title</title>");
          out.print("   </head>");
          out.print("   <body>");
          out.print("     <script type='text/javascript'>function del(dno){var res = window.confirm('确定删除?');if(res){document.location.href='"+ContextPath+"/dept/delete?deptno='+dno}}</script>");
          out.print("       <table border='1px' align='center' width='50%'>");
          out.print("           <tr>");
          out.print("               <th>序号</th>");
          out.print("               <th>部门编号</th>");
          out.print("               <th>部门名称</th>");
          out.print("               <th>操作</th>");
          out.print("           </tr>");
  
          Connection conn = null;
          PreparedStatement pstmt = null;
          ResultSet rs = null;
  
          try {
              conn = DBUtil.getConnection();
              //创建预编译语句对象
              pstmt = conn.prepareStatement("select deptno,dname from dept");
              //执行并获取结果集
              rs = pstmt.executeQuery();
  
              int id = 0;
              while (rs.next()) {
                  String deptno = rs.getString("deptno");
                  String dname = rs.getString("dname");
  
                  out.print("           <tr>");
                  out.print("               <th>"+(++id)+"</th>");
                  out.print("               <th>"+deptno+"</th>");
                  out.print("               <th>"+dname+"</th>");
                  out.print("               <th>");
                  out.print("                   <a href='javascript:void(0)' onclick='del("+deptno+")'>删除</a>");
                  out.print("                   <a href='"+ContextPath+"/dept/edit?deptno="+deptno+"'>修改</a>");
                  out.print("                   <a href='"+ContextPath+"/dept/detail?deptno="+deptno+"'>详细</a>");
                  out.print("               </th>");
                  out.print("           </tr>");
              }
          } catch (SQLException e) {
              e.printStackTrace();
          }finally {
              DBUtil.close(conn, pstmt, rs);
          }
  
          out.print("       </table>");
          out.print("       <hr>");
          out.print("       <a href='"+ContextPath+"/add.html'>增加</a>");
          out.print("   </body>");
          out.print("   </html>");
      }
  
      private void doSave(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          String deptno = request.getParameter("deptno");
          String dname = request.getParameter("dname");
          String loc = request.getParameter("loc");
  
          int count = 0;
  
          Connection conn = null;
          PreparedStatement pstmt = null;
  
          try {
              conn = DBUtil.getConnection();
              pstmt = conn.prepareStatement("insert into dept(deptno, dname, loc) values(?,?,?)");
              pstmt.setString(1, deptno);
              pstmt.setString(2, dname);
              pstmt.setString(3, loc);
              count = pstmt.executeUpdate();
          } catch (SQLException e) {
              e.printStackTrace();
          }finally {
              DBUtil.close(conn,pstmt,null);
          }
  
          if (count > 0) {
              response.sendRedirect(request.getContextPath() + "/dept/list");
          }else
              response.sendRedirect(request.getContextPath() + "/error.html");
      }
  
      private void doEdit(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          response.setContentType("text/html");
          PrintWriter out = response.getWriter();
  
          String ContextPath = request.getContextPath();
  
          out.print("         <!DOCTYPE html>");
          out.print(" <html lang='en'>");
          out.print(" <head>");
          out.print("     <meta charset='UTF-8'>");
          out.print("     <title>Edit</title>");
          out.print(" </head>");
          out.print(" <body>");
          out.print(" <h1>修改部门</h1>");
          out.print(" <hr>");
          out.print(" <form action='"+ContextPath+"/dept/modify' method='post'>");
  
          String deptno = request.getParameter("deptno");
  
          Connection conn = null;
          PreparedStatement pstmt = null;
          ResultSet rs = null;
  
          try {
              conn = DBUtil.getConnection();
              pstmt = conn.prepareStatement("select dname,loc from dept where deptno = ?");
              pstmt.setString(1, deptno);
              rs = pstmt.executeQuery();
              if (rs.next()) {
                  String dname = rs.getString("dname");
                  String loc = rs.getString("loc");
  
                  out.print("                 部门编号<input type='text' name='deptno' value='"+deptno+"' readonly>");
                  out.print("     <br>");
                  out.print("                 部门名称<input type='text' name='dname' value='"+dname+"'>");
                  out.print("     <br>");
                  out.print("                 位置<input type='text' name='loc' value='"+loc+"'>");
                  out.print("     <br>");
              }
          } catch (SQLException e) {
              e.printStackTrace();
          }finally {
              DBUtil.close(conn, pstmt, rs);
          }
  
          out.print("     <input type='submit' name='提交'>");
          out.print(" </body>");
          out.print(" </html>");
      }
  
      private void doDel(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          String deptno = request.getParameter("deptno");
          int count = 0;
  
          Connection conn = null;
          PreparedStatement pstmt = null;
  
          try {
              conn=DBUtil.getConnection();
              //开启事务
              conn.setAutoCommit(false);
              pstmt=conn.prepareStatement("delete from dept where deptno=?");
              pstmt.setString(1, deptno);
              //返回值为影响了数据库中的记录量
              count = pstmt.executeUpdate();
              //事务提交
              conn.commit();
          } catch (SQLException e) {
              //回滚
              if(conn==null){
                  try {
                      conn.rollback();
                  } catch (SQLException ex) {
                      throw new RuntimeException(ex);
                  }
              }
              e.printStackTrace();
          }finally {
              DBUtil.close(conn,pstmt,null);
          }
  
          if(count>0){
              response.sendRedirect(request.getContextPath()+"/dept/list");
          }else
              response.sendRedirect(request.getContextPath()+"/error.html");
      }
  
      private void doModify(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          String deptno = request.getParameter("deptno");
          String dname = request.getParameter("dname");
          String loc = request.getParameter("loc");
  
          int count = 0;
  
          Connection conn = null;
          PreparedStatement pstmt = null;
  
          try {
              conn = DBUtil.getConnection();
              pstmt = conn.prepareStatement("update dept set dname=?, loc=? where deptno=?");
              pstmt.setString(1, dname);
              pstmt.setString(2, loc);
              pstmt.setString(3, deptno);
              count = pstmt.executeUpdate();
          } catch (SQLException e) {
              e.printStackTrace();
          }finally {
              DBUtil.close(conn,pstmt,null);
          }
  
          if (count > 0) {
              response.sendRedirect(request.getContextPath()+"/dept/list");
          }else
              response.sendRedirect(request.getContextPath()+"/error.html");
      }
  
      private void doDetail(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
          response.setContentType("text/html");
          PrintWriter out = response.getWriter();
  
          out.print("         <!DOCTYPE html>");
          out.print(" <html lang='en'>");
          out.print(" <head>");
          out.print("     <meta charset='UTF-8'>");
          out.print("     <title>Detail</title>");
          out.print(" </head>");
          out.print(" <body>");
          out.print("  <h1>部门详情</h1><hr>");
  
          String deptno = request.getParameter("deptno");
  
          Connection conn = null;
          PreparedStatement pstmt = null;
          ResultSet rs = null;
  
          try {
  
              conn = DBUtil.getConnection();
              pstmt = conn.prepareStatement("select dname,loc from dept where deptno = ?");
              //替换字符
              pstmt.setString(1, deptno);
              rs = pstmt.executeQuery();
  
              if (rs.next()) {
                  String dname = rs.getString("dname");
                  String loc = rs.getString("loc");
  
                  out.print("                 部门编号: "+deptno+"<br>");
                  out.print("                 部门名称: "+dname+"<br>");
                  out.print("                 部门位置: "+loc+"<hr>");
              }
  
          } catch (SQLException e) {
              e.printStackTrace();
          }finally {
              DBUtil.close(conn, pstmt, rs);
          }
  
          out.print(" <input type='button' value='back' onclick='window.history.back()' />");
          out.print(" </body>");
          out.print(" </html>");
      }
  
  }
  
  ```

### 关于 B/S 结构系统的会话机制(session 机制)

- 会话
  - 用户打开浏览器开始, 经过一系列操作, 最终关闭浏览器, 整个过程称为一次会话, 服务端对应的一个 java 对象: session
  - 一次会话包含多个次请求
  - Servlet 规范中的会话类: HttpSession
  - 主要作用: 保存会话状态, 如用户的登录状态等
    - 服务器无从得知浏览器是否关闭, 而是通过会话来判断用户的登录状态
    - 一个浏览器实例对应一个会话
    - 服务器通过会话 id 管理客户端状态
  - request(请求域)< session(会话域)< application(应用域)
    - 这三个域对象的公共方法
      - setAttribute（向域当中绑定数据）
      - getAttribute（从域当中获取数据）
      - removeAttribute（删除域当中的数据）
  
- Cookie

  - 是 HTTP 协议中用于在客户端存储小型数据的核心技术

  - 由服务器发送到用户浏览器并保存的小型文本文件, 后续向该服务器发送请求中会自动携带这些数据

  - ```mermaid
    sequenceDiagram
        participant Client
        participant Server
        
        Client->>Server: 首次请求（无 Cookie）
        Server->>Client: HTTP响应头包含：Set-Cookie: user_id=12345
        Client->>Server: 后续请求（自动包含：Cookie: user_id=12345）
        Server->>Client: 根据Cookie提供个性化响应
    
    ```

  - Cookie 数据由键值对形式存储

    HTTP 协议规定: 任何一个 Cookie 都由 name 与 value 组成, 且都为字符串类型

  - Cookie 与域名绑定, 只对创建它的域名有效

  - 每个域名下最多约 50 个 Cookie，总大小约 4KB

  - Cookie 由属性控制其行为:

    ```http
    Set-Cookie: user_token=abc123xyz; 
      Domain=.example.com; 
      Path=/profile; 
      Expires=Wed, 21 Oct 2022 07:28:00 GMT; 
      Max-Age=3600; 
      Secure; 
      HttpOnly; 
      SameSite=Lax
    ```

    

    | 属性                | 说明                 | 默认值            |
    | ------------------- | -------------------- | ----------------- |
    | **Domain**          | 指定 Cookie 生效的域名 | 当前主机名        |
    | **Path**            | 指定 Cookie 生效的路径 | 当前路径          |
    | **Expires/Max-Age** | 设置过期时间         | 会话结束过期      |
    | **Secure**          | 仅通过 HTTPS 传输      | 无此限制          |
    | **HttpOnly**        | 禁止 JavaScript 访问   | 可通过 JS 访问      |
    | **SameSite**        | 控制跨站发送行为     | Lax（现代浏览器） |

  - 生命周期:

    - 创建: 服务器通过 Set-Cookie 响应头创建 Cookie
    - 存储: 浏览器按照域名分类存储 Cookie
    - 发送: 浏览器自动在满足条件的请求中添加 Cookie 头
    - 过期: 由设定时间自动清理
      - 会话 Cookie: 浏览器关闭时清理(保存在内存中)
      - 持久 Cookie: 达到指定时间(Expires/Max-Age)后删除(保存在硬盘中)

  - java 的 Servlet 中提供的支持:

    - Cookie 类

      - 设置有效时间: cookie.setMaxAge(time), 单位秒

        没有设置时间则默认保存在浏览器运行内存中, time > 0 则保存在硬盘中

      - 设置 Cookie 路径: cookie.setPath(“/...”)

      - 接受 Cookie:

        ```java
        Cookie[] cookies = request.getCookies(); // 这个方法可能返回null
        if(cookies != null){
            for(Cookie cookie : cookies){
                // 获取cookie的name
                String name = cookie.getName();
                // 获取cookie的value
                String value = cookie.getValue();
            }
        }
        ```

        

    - response.addCookie(cookie): 将 cookie 响应至浏览器

  - Cookie 的 Path

    - 若 Cookie 未设置路径, 则默认 Path 为 http://.../cookie 以及它的子路径
    - 也就是说, 该路径以及它的子路径的请求都会将 cookie 发送至服务器

  - 作用: 在无状态的 HTTP 协议上实现有状态交互

  - 应用:

    - 购物车
    - 免登录

- session 实现原理:

  - JSESSIONID, 以 Cookie 的形式保存在浏览器中(会话 Cookie)
  - session 列表是一个 Map, key: sessionid, value: session 对象
  - 第一次请求, 服务器生成 session 对象, 同时生成 id
  - 第二次请求以及之后,
  - 自动将浏览器内存中的 id 发送给服务器，服务器根据 id 查找 session 对象
  - 关闭浏览器，内存消失，cookie 消失，sessionid 消失，会话等同于结束

- Cookie 禁用:

  - Cookie 禁用是指浏览器拒绝接收服务器发送的 Cookie
  - 此时 session 丢失, 每次请求都会获取一个新的 session
  - Cookie 禁用, session 机制可通过 URL 重写机制实现, 但会提高开发成本, 因此大部分网站在禁用 Cookie 后都无法使用

- session 对象的销毁:

  ```Java
  session.invalidate();
  ```

  

