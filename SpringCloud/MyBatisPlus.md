## 环境配置

- 配置数据库参数

- 引入依赖

  ```xml
  <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.5.3.1</version>
  </dependency>
  <!--
  该依赖包含了MyBatis依赖
  -->
  ```

- 定义 Mapper

  MyBatisPlus 提供了一个基础的 BaseMapper 接口, 已实现单表的 CRUD 操作

  定义接口 Mapper 继承 BaseMapper 接口

## PO,VO,DTO

Persistent Object, 持久层对象, 即 **数据库中与某张表对应的 Java 对象(表字段的封装), 其属性与表中字段对应**

- MyBatisPlus 根据 PO 实体的信息来推断出表中信息, 从而生成 sql 的

- 将 PO 实体的类名驼峰转下划线作为表名

- 将 PO 实体的所有变量名驼峰转下划线作为表的字段名, 并根据变量类型推断字段类型

- 将名为 id 的字段转为主键

- 与表名及字段名一一对应

- 存在生命周期

- 包含getter与setter

- ```java
  // 假设数据库中有一张 book 表
  public class BookPO {
      private Long id;         // 对应数据库表的 id (主键)
      private String isbn;     // 对应数据库表的 isbn 字段
      private String title;    // 对应数据库表的 title 字段
      private String author;   // 对应数据库表的 author 字段
      private BigDecimal price; // 对应数据库表的 price 字段 (进价)
      private Integer stock;   // 对应数据库表的 stock 字段 (库存)
  
      // ... getter 和 setter 方法 ...
  }
  ```

  

Data Transfer Object, 数据传输对象,其主要目的是**减少网络传输数据量**与**隐藏内部细节**

- 按需定制

- 扁平化结构

- 存在getter与setter

- ```java
  public class BookDTO {
      private Long id;         // 图书ID
      private String title;    // 书名
      private String author;   // 作者
      private BigDecimal price; // 售价 (可能和PO中的进价不同)
  
      // ... getter 和 setter 方法 ...
  }
  ```

View Object, 视图对象, 侧重于视图层的展示,用于**格式化和封装数据**

- 面向展示

- 可以包含衍生数据

- 只读,即仅存在getter

- ```java
  public class BookVO {
      private Long id;
      private String title;
      private String author;
      private String formattedPrice; // 格式化后的价格，如 "¥99.00"
      private String publishDateStr; // 格式化后的出版日期，如 "2023-10-27"
      private String stockStatus;    // 库存状态，如 "有货" 或 "缺货"
  
      // ... getter 方法 ...
  }
  ```

BeanUtil.copyProperties():使用反射机制,将一个Java对象的属性值按照属性名复制到另一个Java对象

## 注解

### @TableName

用于 pojo, 标识实体类对应的表

```java
@TableName("user")
/*
注解属性
value:表名
schema:数据库名
keepGlobalPrefix:false(是否使用全局的tablePrefix,即表前缀)
resultMap:xml中resultMap的id
autoResultMap:false(是否自动构建resultMap并使用)
*/
public class User {
    private Long id;
    private String name;
}
```

### @TableId

用于 pojo, 标识主键

```java
@TableName("user")
public class User {
    @TableId
    /*
注解属性
value:表名
type:IdType.NONE(主键策略)
*/
    private Long id;
    private String name;
}
```

#### 主键策略

IdType.类型:

- AUTO: 自增
- NONE: 未设置主键类型, 在注解里表示跟随全局, 在全局里近似于 INPUT
- INPUT: 自行设置
- ASSIGN_ID: 分配 ID, 使用雪花算法, 若全局未配置则默认该策略
- ASSIGN_UUID: 分配 UUID

### @TableField

普通字段注解, 用于一下情况

- 成员变量名与数据库字段名不一致
- 不符合 JavaBean 命名规范, 如以 is 开头
- 与数据库关键字冲突, 使用反引号(``)为字段名转义

```java
@TableName("user")
public class User {
    @TableId
    private Long id;
    private String name;
    private Integer age;
    @TableField(is_married")
    private Boolean isMarried;
    @TableField("`concat`")
/*
注解属性
value:数据库字段名
exist:true(是否为数据库字段,若为false,则不会查询该字段的数据)
condition:指定字段表达式where
update:自定义更新逻辑
insertStrategy:控制在insert时是否包含于insert的字段
updateStrategy:控制在update时如何处理字段的值
whereStrategy:控制where子句如何处理字段的值
fill:字段自动填充策略
select:是否进行select查询
keepGlobalFormat:是否使用保持全局的format进行处理
jdbcType:JDBC类型
typeHandle:类型处理器
mumericScale:指定小数保留
*/
    private String concat;
}
```

## 配置

大多数配置都有默认值, 但部分配置项没有默认值

- 实体类的别名扫描包

- 全局 id 类型

  ```yaml
  mybatis-plus:
    type-aliases-package: com.itheima.mp.domain.po
    global-config:
      db-config:
        id-type: auto # 全局id类型为自增长
  ```

- 支持手写 sql, 其读取文件位置可自行配置(因为 mp 更擅长单表操作，如果是复杂的多表操作，还是要通过手写的方式):

  ```yml
  mybatis-plus:
  	mapper-locations: "classpath*:/mapper/**/*.xml" # Mapper.xml文件地址，当前这个是默认值。
  # 指向resource文件下的对应地址
  # classpath*:与classpath:是有区别的，前者是所有模块下面的mapper文件夹
  # 两个*代表mapper下面有子文件也会被扫描到，默认不需要配置
  ```

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.itheima.mp.mapper.UserMapper">
  
      <select id="queryById" resultType="User"> <!--上述别名扫描包type-aliases-package的作用就是这里不用写完整类路径-->
          SELECT * FROM user WHERE id = #{id}
      </select>
  </mapper>
  ```

  使用:

  ```java
  @Test
  void testQuery() {
      User user = userMapper.queryById(1L);
      // 搜索指定id的sql语句
      System.out.println("user = " + user);
  }
  ```


## 条件构造器

Wrapper, 用于构造复杂的查询条件(where), 其继承结构如下

![image-20251105150532895](./MyBatisPlus.assets/image-20251105150532895.png)

### QueryWrapper

```java
// select id,username,info,balance from user where username like '%o%' and balance >= 1000
void testQueryWrapper(){
    // 创建构造器 
    QueryWrapper<User> queryWrapper = new QueryWrapper<User>()
        .select("id","username","info","balance") // 查询字段
        .like("username","o") // 模糊查询
        .ge("balance",1000); // 大于等于
    // 查询结果
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}

// update user set balance = 2000 where username = 'Jack'
void testQueryWrapper(){
    // 创建构造器
    QueryWrapper<User> queryWrapper = new QueryWrapper<User>()
        .eq("username", "jack");
    // 更新数据,非null数据都会作为set语句中的字段
    User user = new User();
    user.setBalance(2000);
    // set是直接赋值的,无法动态更新数据,如在源数据上进行增删操作
    userMapper.update(user, queryWrapper);
}
```

### UpdateWrapper

set 基于字段现有值时, 使用 UpdateWrapper 中的 setSql:

```java
// update user set balance = balance - 200 where id in (1,2,4)
void testUpdateWrapper(){
        List<Long> ids = List.of(1L,2L,4L);
        // 创建构造器
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<User>()
                .setSql("balance = balance - 200")
                . in("id", ids);
        // 更新
        // 第一个参数表示不需要依赖实体对象进行更新,完全依赖UpdateWrapper中设置的条件和setSql
        userMapper.update(null, updateWrapper);
    }
```

### LambdaQueryWrapper/LambdaUpdateWrapper

为防止字符串魔法值(在代码中硬编码字符串常量, 使得字段名修改时需要重新修改代码), MyBatisPlus 提供了一套基于 Lambda 的 Wrapper

```java
void testLambdaQueryWrapper(){
    //  创建构造器
    LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<User>()
        .select(User::getId, User::getUsername, User::getInfo,User::getBalance)
        .like(User::getUsername, "jack")
        .ge(User::getBalance, 2000);
    /*
        使用wrapper.lambda
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<>();
        userQueryWrapper.lambda()
                .select(User::getId, User::getUsername, User::getInfo,User::getBalance)
                .like(User::getUsername, "jack")
                .ge(User::getBalance, 2000);
        */
    // 查询
    List<User> users = userMapper.selectList(lambdaQueryWrapper);
    users.forEach(System.out::println);
}
```

## 自定义 sql

Sql语句最好统一维持在持久层而非业务层

```java
// UserMapper
public interface UserMapper extends BaseMapper<User> {
    @Update("update user set balance = balance - #{money} ${ew.customSqlSegment}")
    void deductBalanceByIds(@Param("money") int money, @Param("ew") LambdaQueryWrapper<User> wrapper);
}
// ${ew.customSqlSegment}
// ew指向传入参数
// customSqlSegment是MyBatis-Plus框架中自定义SQL方法,以动态生成where条件子句
// ${}而非#{},表示字符串替换而非参数预编译
```

```java
// Wrapper
void testCustomWrapper(){
        // 自定义查询条件
        List<Long> ids = List.of(1L,2L,4L);
        LambdaQueryWrapper<User> user = new LambdaQueryWrapper<User>()
                .in(User::getId, ids);
        // 传递wrapper
        userMapper.deductBalanceByIds(200,user);
    }
```

多表关联实现

```java
// Wrapper
void testCustomJoinWrapper(){
        QueryWrapper<User> userQueryWrapper = new QueryWrapper<User>()
                .in("u.id",List.of(1L,2L,4L))
                .eq("a.city","北京");
        List<User> users = userMapper.queryUserByWrapper(userQueryWrapper);
        users.forEach(System.out::println);
    }
```

```java
// Mapper
@Select("select u.* from user u inner join address a on u.id = a.user_id ${ew.customSqlSegment}")
    List<User> queryUserByWrapper(@Param("ew") QueryWrapper<User> wrapper);
```

## Service接口(IService,ServiceImpl)

MyBatisPlus不仅提供了BaseMapper接口,还提供了通用的Service接口及默认实现,封装一些常用的Service方法

### CRUD

- save:新增单个元素
- saveBatch:批量新增
- saveOrUpdate:根据id判断,若数据存在则更新,不存在增添
- saveOrUpdateBatch:批量新增或修改
- removeById:根据id删除
- removeByIds:批量根据id删除
- removeByMap:根据map中的键值对为条件删除
- remove(Wrapper<T>):根据Wrapper条件删除
- updateById:根据id修改
- update(Wrapper<T>):根据updateWrapper修改,Wrapper中包含set和where部分
- update(T,Wrapper<T>):按照T内的数据修改与Wrapper匹配到的数据
- updateBatchById:根据id批量修改
- getById:根据id查询一条数据
- getOne(Wrapper<T>):根据wrapper查询一条数据
- getBaseMapper:获取Service内的BaseMapper实现,某些时候需要直接调用Mapper内的自定义SQL时可以用这个方法获取到Mapper
- listByIds:根据id批量查询
- list(Wrapper<T>):根据Wrapper条件查询多条数据
- list():查询所有
- count():统计所有数量
- count(Wrapper<T>):统计符合Wrapper条件的数据数量
- getBaseMapper():在service中获取对应的自定义Mapper

### 基本使用

**Service层**

```java
// IUserService
public interface IUserService extends IService<User> {
    // 继承IService接口以拓展自定义方法
    void deductBalance(Long id,Integer money);
}
```

```java
// UserServiceImpl
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {
	// 继承了IService的实现类
    public void deductBalance(Long id, Integer money) {
        User user = getById(id);
        if (user == null || user.getStatus() == 2) {
            throw new RuntimeException("用户异常");
        }
    
        if (user.getBalance() < money) {
            throw new RuntimeException("余额不足");
        }

        baseMapper.deductBalanceByIds(money, new LambdaQueryWrapper<User>().in(User::getId, id));
    }
}
```

**Controller层**

```java
@Api(tags = "用户管理接口")
@RequiredArgsConstructor
@RestController
@RequestMapping("users")
public class UserController {

    private final IUserService userService;

    @PostMapping
    @ApiOperation("新增用户")
    public void saveUser(@RequestBody UserFormDTO userFormDTO) {
        // 转换 UserFormDTO --> User
        User user = BeanUtil.copyProperties(userFormDTO, User.class);
        userService.save(user); // 新增单个元素
    }

    @DeleteMapping("/{id}")
    @ApiOperation("删除用户")
    public void RemoveUserById(@PathVariable("id") Long userId) {
        userService.removeById(userId);
    }

    @GetMapping("/{id}")
    @ApiOperation("根据Id查询用户")
    public UserVO getUserById(@PathVariable("id") Long userId) {
        User user = userService.getById(userId);
        return BeanUtil.copyProperties(user, UserVO.class);
    }

    @GetMapping
    @ApiOperation("根据id集合查询用户")
    public List<UserVO> getByIds(@RequestParam("ids") List<Long> userIds) {
        List<User> users = userService.listByIds(userIds);
        return BeanUtil.copyToList(users, UserVO.class);
    }

    @PutMapping("/{id}/deduction/{money}")
    @ApiOperation("根据id扣减余额")
    public void deduction(@PathVariable("id") Long userId, @PathVariable("money") Integer money) {
        userService.deductBalance(userId, money);
    }
}
```

### Lambda

IService提供了Lambda功能(lambdaQuery,lambdaUpdate)以简化复杂查询及更新功能

```java
@GetMapping("/list")
    @ApiOperation("根据id集合查询对象")
    public List<UserVO> queryUsers(UserQueryDTO userQueryDTO) {
        // 获取查询条件
        String Username = userQueryDTO.getName();
        Integer status = userQueryDTO.getStatus();
        Integer minBalance = userQueryDTO.getMinBalance();
        Integer maxBalance = userQueryDTO.getMaxBalance();
/*        // 创建Wrapper
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<User>()
                .like(Username != null,User::getUsername,Username)
                .eq(status != null,User::getStatus, status)
                .ge(minBalance != null,User::getBalance,minBalance)
                .le(maxBalance != null,User::getBalance,maxBalance);
        // 查询
        List<User> users = userService.list(queryWrapper);*/
        // 基于Lambda查询
        List<User> users = userService.lambdaQuery()
                .like(Username != null, User::getUsername, Username)
                .eq(status != null, User::getStatus, status)
                .ge(minBalance != null, User::getBalance, minBalance)
                .le(maxBalance != null, User::getBalance, maxBalance)
                .list();
        // 类型转换
        return BeanUtil.copyToList(users, UserVO.class);
    }
```

- 基于Lambda查询,最后的list()表示查询的结果需要是一个list集合,这里还有其他可选方法:
  - one():最多一个结果
  - list():返回集合结果
  - count():返回计数结果
- 基于Lambda对deductBalance进行的优化:

```java
public void deductBalance(Long id, Integer money) {
        // 根据id查询指定用户
        User user = getById(id);
        // 判断用户状态
        if (user == null || user.getStatus() == 2) {
            throw new RuntimeException("用户异常");
        }
        if (user.getBalance() < money) {
            throw new RuntimeException("余额不足");
        }
        // 调用Mapper层方法
//        baseMapper.deductBalanceByIds(money, new LambdaQueryWrapper<User>().in(User::getId, id));
        // 基于Lambda更新
        int remainBalance = user.getBalance() - money;
        lambdaUpdate().set(User::getBalance, remainBalance)
                .set(remainBalance == 0, User::getStatus, 2) // 实现动态更新status
                .eq(User::getId, id)
                .eq(User::getBalance, user.getBalance()) // 乐观锁,当两次更新同时进行时,允许最先执行的成功,后者失败
                .update();
    }
```

### 批处理

```java
// 逐条处理
void testSaveOneByOne() {
    long b = System.currentTimeMillis();
    for (int i = 1; i <= 100000; i++) {
        userService.save(buildUser(i));
    }
    long e = System.currentTimeMillis();
    System.out.println("耗时：" + (e - b));
}
```

每一次提交都是一次网络请求,因此大量数据逐条提交耗时极长

```java
// 批量处理
void testSaveBatch() {
    List<User> list = new ArrayList<>(1000);
    for (int i = 1; i <= 100000; i++) {
        list.add(buildUser(i));
        if (i % 1000 == 0) {
            userService.saveBatch(list);
            list.clear();
        }
    }
}
```

MyBatisPlus的批处理,即将多条数据封装为一个集合(**集合中的每一条数据为一条sql语句**),一次网络请求中发送多条语句,然后由Mysql逐条执行

实质上,若想得到最佳性能,应将多条sql合并为一条sql语句:

```sql
INSERT INTO user ( username, password, phone, info, balance, create_time, update_time )
VALUES 
(user_1, 123, 18688190001, "", 2000, 2023-07-01, 2023-07-01),
(user_2, 123, 18688190002, "", 2000, 2023-07-01, 2023-07-01),
(user_3, 123, 18688190003, "", 2000, 2023-07-01, 2023-07-01),
(user_4, 123, 18688190004, "", 2000, 2023-07-01, 2023-07-01);
```

Mysql提供了`rewriteBatchedStatements`参数,以重写批处理的statement语句,该参数的默认值为false,需要在连接的配置文件中将其设置为true:

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/mp?rewriteBatchedStatements=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: MySQL123
```

## 扩展功能

### 静态工具Db

有时Service之间会相互调用,从而出现循环依赖问题,MyBatisPlus提供一个静态工具类:`Db`,其中一些静态方法与IService中方法签名基本一致,可用于实现CRUD功能

### 逻辑删除

对于一些比较重要的数据,常常采用逻辑删除方案:

- 在表中添加一个字段用于表示是否被删除
- 删除数据时只将该字段标记置为true
- 在查询时过滤标记为true的数据

然而,一但采用逻辑删除,所有查询和删除的逻辑都要变化,为解决该问题,MyBatisPlus添加了对逻辑删除的支持,**只有MyBatisPlus生成的sql语句才支持自动的逻辑删除,自定义sql需要自己手动处理逻辑删除**

```sql
# 添加字段
alter table address add deleted bit default b'0' null comment '逻辑删除';
```

```YAML
# 配置文件
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted # 全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不添加po中的属性)
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

逻辑删除本身具有很大问题:

- 数据滞留堆积会导致查询效率降低
- 所有查询都要进行逻辑判断,影响效率

因而一般并不采用逻辑删除功能,通常使用**数据迁移**或**多库备份**等方案进行替代,类似创建一张表专用于存储删除的数据

### 通用枚举

MyBatisPlus提供了一个处理枚举类型的类型转换器,可以**将枚举类型与数据库类型自动转换**

创建枚举类型:

```java
package com.itheima.mp.enums;

import com.baomidou.mybatisplus.annotation.EnumValue;
import lombok.Getter;

@Getter
public enum UserStatus {
    NORMAL(1, "正常"),
    FREEZE(2, "冻结")
    ;
    @EnumValue // @EnumValue注解由MyBatisPlus提供,注解标注枚举属性,即告诉MyBatisPlus哪个属性作为数据库字段值
    private final int value;
    @JsonValue // 该注解用于标注JSON序列化时展示的字段,即返回时显示的内容
    private final String desc;
    UserStatus(int value, String desc) {
        this.value = value;
        this.desc = desc;
    }
}
```

配置枚举处理器(旧版本需要)

```yaml
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
```

### JSON类型处理器

数据库表中定义的JSON类型数据,在java中用String类型接收(**MyBatis会对String与JSON类型做相互转换**),使得属性读取极为不便

MyBatisPlus提供了很多特殊类型字段的类处理器,解决特殊字段类型与数据库类型转换的问题,例如JSON可以用`JacksonTypeHandler`处理器

```java
package com.itheima.mp.domain.po;

import lombok.Data;

@Data
public class UserInfo {
    private Integer age;
    private String intro;
    private String gender;
}
// Jackson需要无参构造方法来实例化对象,然后通过反射或setter方法填充字段值,若没有无参构造方法则无法添加这个实例
```

- 将User类中的info字段类型修改为UserInfo,并加上@TableField(typeHandler = JacksonTypeHandler.class)注解
- 在User类上加上@TableName(value = “user”, autoResultMap = true)注解,声明自动映射

## 插件功能

MyBatisPlus提供了很多插件功能:

- PaginationInnerInterceptor : 自动分页
- TenantLineInnerInterceptor : 多租户
- DynamicTableNameInnerInterceptor : 动态表名
- OptimisticLockerInnerInterceptor : 乐观锁
- IllegalSQLInnerInterceptor : sql性能规范
- BlockAttackInnerInterceptor : 防止全表更新与删除
- 定义顺序:
  - 多租户,动态表名
  - 分页,乐观锁
  - sql性能规范,防止全表更新与删除

### 分页插件

在未引入分页插件情况下,MyBatisPlus不支持分页功能,Iservice与BaseMapper中的分页方法都无法正常起效

配置分页插件:

```java
package com.itheima.mp.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MybatisConfig {

    @Bean
    // 总拦截器,拦截sql语句的执行,再拓展功能
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        // 初始化核心插件
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 添加分页插件
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

```xml
<!--
在3.5.9版本中，分页功能依赖的 `jsqlparser` 被单独拆成了 `mybatis-plus-jsqlparser` 包,得手动添加这个依赖
-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-jsqlparser</artifactId>
    <version>3.5.9</version>
    <!-- 确保版本和 MyBatis Plus 主包一致 -->
</dependency>
```

```java
@Test
void testPageQuery() {
    // 1.分页查询，new Page()的两个参数分别是：页码、每页大小
    Page<User> p = userService.page(new Page<>(2, 2));
    // 2.总条数
    System.out.println("total = " + p.getTotal());
    // 3.总页数
    System.out.println("pages = " + p.getPages());
    // 4.数据
    List<User> records = p.getRecords();
    records.forEach(System.out::println);
    
    // 这里用到了分页参数，Page，即可以支持分页参数，也可以支持排序参数。常见的API如下:
    int pageNo = 1, pageSize = 5;
    // 分页参数
    Page<User> page = Page.of(pageNo, pageSize);
    // 排序参数, 通过OrderItem来指定
    page.addOrder(new OrderItem("balance", false));

    userService.page(page);
}
```

