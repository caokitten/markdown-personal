### Controller接收参数

- 对于前端的批量操作,传入多个参数,服务端的接收:

  - 通过数组接收:

    ```java
    /**
    * 批量删除员工
    */
    @DeleteMapping
    public Result delete(Integer[] ids){
        log.info("批量删除部门: ids={} ", Arrays.asList(ids));
        return Result.success();
    }
    ```

  - 通过集合接收:

    ```java
    /**
    * 批量删除员工
    */
    @DeleteMapping
    public Result delete(@RequestParam List<Integer> ids){
        log.info("批量删除部门: ids={} ", ids);
        empService.deleteByIds(ids);
        return Result.success();
    }
    //对于集合类型,默认情况下不会自动绑定请求参数,必须通过@RequestParam显式声明
    ```


### 自定义结果集

- ```xml
  <!--自定义结果集ResultMap-->
  <!--用于对查询结果进行手动封装-->
  <resultMap id="empResultMap" type="com.itheima.pojo.Emp">
      <id column="id" property="id" />
      <result column="username" property="username" />
      <result column="password" property="password" />
      <result column="name" property="name" />
      <result column="gender" property="gender" />
      <result column="phone" property="phone" />
      <result column="job" property="job" />
      <result column="salary" property="salary" />
      <result column="image" property="image" />
      <result column="entry_date" property="entryDate" />
      <result column="dept_id" property="deptId" />
      <result column="create_time" property="createTime" />
      <result column="update_time" property="updateTime" />
  
      <!--封装exprList-->
      <!--嵌套-->
      <collection property="exprList" ofType="com.itheima.pojo.EmpExpr">
          <id column="ee_id" property="id"/>
          <result column="ee_company" property="company"/>
          <result column="ee_job" property="job"/>
          <result column="ee_begin" property="begin"/>
          <result column="ee_end" property="end"/>
          <result column="ee_empid" property="empId"/>
      </collection>
  </resultMap>
  
  <!--sql语句-->
  <!--根据ID查询员工的详细信息,这里使用resultMap而非resultType-->
  <select id="getById" resultMap="empResultMap">
      select e.*,
          ee.id ee_id,
          ee.emp_id ee_empid,
          ee.begin ee_begin,
          ee.end ee_end,
          ee.company ee_company,
          ee.job ee_job
      from emp e left join emp_expr ee on e.id = ee.emp_id
      where e.id = #{id}
  </select>
  ```

  

