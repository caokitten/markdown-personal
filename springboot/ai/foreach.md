## 🤖 Assistant

好的，我们来系统、深入地详解 MyBatis XML 中的 `<foreach>` 标签。这是 MyBatis 动态 SQL 中最强大、最灵活的标签之一，掌握它对于处理集合、数组等参数至关重要。

### `<foreach>` 的核心作用

一句话概括：**`<foreach>` 标签用于迭代一个集合（如 List, Set, Array, Map），并根据集合内容动态地生成 SQL 语句的一部分。**

它最典型的应用场景有两个：

1. **构建动态 `IN` 子句**：例如 `WHERE id IN (1, 2, 3, ...)`，其中括号里的 ID 数量是可变的。
2. **执行批量插入 (Batch Insert)**：一次性向数据库插入多条记录，例如 `INSERT INTO my_table (col1, col2) VALUES (?, ?), (?, ?), (?, ?)...`。

---

### `<foreach>` 标签的属性详解

这是 `<foreach>` 的基本语法结构，我们来逐个拆解它的属性：

```xml
<foreach collection="collection" item="item" index="index" open="open" close="close" separator="separator">
    <!-- 循环体内容 -->
</foreach>
```

| 属性 | 解释 | 是否必须 | 示例说明 |
| :--- | :--- | :--- | :--- |
| `collection` | **要迭代的集合对象**。这是最重要的属性，它的值是你传入 Mapper 方法的参数名。 | **是** | 如果你的方法是 `List<User> findUsers(List<Integer> ids)`，那么 `collection` 就应该是 `"ids"`。如果使用了 `@Param` 注解，如 `@Param("userIds") List<Integer> ids`，那么 `collection` 就应该是 `"userIds"`。 |
| `item` | **当前迭代元素的别名**。在循环体内部，你可以使用这个别名来引用集合中的每一个元素。 | **是** | 如果 `collection` 是一个 `List<Integer>`，那么 `item` 就代表每一个 `Integer` 值。你可以写 `#{item}`。如果是一个 `List<User>`，`item`就代表每一个 `User` 对象，你可以写 `#{item.id}`、`#{item.name}`。 |
| `index` | **当前迭代的索引或键的别名**。在循环体内部，你可以使用这个别名来引用当前元素的位置。 | 否 | 如果 `collection` 是 List 或 Array，`index` 的值是 `0, 1, 2, ...`。如果 `collection` 是 Map，`index` 的值是 Map 的 key。 |
| `open` | **在整个循环内容开始前，要拼接的字符串**。 | 否 | 对于 `IN` 子句，通常是 `(`。 |
| `close` | **在整个循环内容结束后，要拼接的字符串**。 | 否 | 对于 `IN` 子句，通常是 `)`。 |
| `separator` | **每次迭代之间要插入的分隔符**。MyBatis 很智能，它不会在最后一次迭代后添加这个分隔符。 | 否 | 对于 `IN` 子句或批量插入的值列表，通常是 `,`。 |

**一个形象的比喻：**

想象你在用代码拼接一个字符串列表 `["Apple", "Banana", "Cherry"]`，你想得到最终结果 `('Apple','Banana','Cherry')`。

* `collection`: 就是那个 `["Apple", "Banana", "Cherry"]` 列表。
* `item`: 就是循环中依次出现的 "Apple", "Banana", "Cherry"。
* `open`: 就是最开始的 `(`。
* `close`: 就是最后面的 `)`。
* `separator`: 就是每个水果之间的 `,`。

`<foreach>` 标签就是帮你自动完成这个拼接过程的工具。

---

### 实战案例

#### 案例一：构建动态 `IN` 子句（最常用）

**需求**：根据一个给定的 ID 列表，查询所有匹配的用户。

**1. Mapper 接口 (Java)**

```java
import org.apache.ibatis.annotations.Param;
import java.util.List;

public interface UserMapper {
    // 使用 @Param 注解给参数命名，这是最佳实践
    List<User> findUsersByIds(@Param("userIds") List<Integer> ids);
}
```
> **注意**：当 Mapper 方法有多个参数时，强烈推荐使用 `@Param` 注解来明确指定参数在 XML 中应该使用的名字。如果只有一个集合参数，MyBatis 默认会使用 `"list"` 或 `"collection"` 作为 `collection` 属性的值，但这会降低代码可读性。

**2. Mapper XML**

```xml
<select id="findUsersByIds" resultType="com.example.User">
    SELECT * FROM user
    WHERE id IN
    <foreach collection="userIds" item="userId" open="(" separator="," close=")">
        #{userId}
    </foreach>
</select>
```

**解析：**
* `collection="userIds"`: 对应 Mapper 方法中 `@Param("userIds")` 的命名。
* `item="userId"`: 在循环中，我们给每个 ID 起了个别名叫 `userId`。
* `open="("`: 在拼接的开头加上 `(`。
* `separator=","`: 在每个 ID 之间加上 `,`。
* `close=")"`: 在拼接的结尾加上 `)`。
* `#{userId}`: 引用当前循环中的 ID 值。

**假设调用时传入的 `ids` 列表是 `[101, 102, 105]`，MyBatis 将会生成如下 SQL：**

```sql
SELECT * FROM user WHERE id IN ( ? , ? , ? )
-- 实际执行时，参数会被替换为：
-- SELECT * FROM user WHERE id IN (101, 102, 105)
```

#### 案例二：批量插入 (Batch Insert)

**需求**：一次性向 `user` 表插入多条用户记录。

**1. Mapper 接口 (Java)**

```java
import org.apache.ibatis.annotations.Param;
import java.util.List;

public interface UserMapper {
    // 传入一个 User 对象的列表
    void insertUsers(@Param("userList") List<User> users);
}
```

**2. Mapper XML**

```xml
<insert id="insertUsers">
    INSERT INTO user (name, email, age)
    VALUES
    <foreach collection="userList" item="user" separator=",">
        (#{user.name}, #{user.email}, #{user.age})
    </foreach>
</insert>
```

**解析：**
* `collection="userList"`: 对应方法参数的名字。
* `item="user"`: 每次循环的元素是一个 `User` 对象，我们给它起别名为 `user`。
* `separator=","`: 每个 `(...)` 值块之间用逗号隔开。
* `#{user.name}`: 在循环体内，通过 `.` 来访问 `user` 对象的属性。
* 这里我们省略了 `open` 和 `close`，因为 `VALUES` 关键字后面直接就是值列表。

**假设调用时传入的 `users` 列表包含三个 `User` 对象，MyBatis 将会生成如下 SQL：**

```sql
INSERT INTO user (name, email, age)
VALUES
    ( ? , ? , ? ) , ( ? , ? , ? ) , ( ? , ? , ? )
-- 实际执行时，参数会被替换为：
-- INSERT INTO user (name, email, age) VALUES ('Alice', 'a@a.com', 25), ('Bob', 'b@b.com', 30), ('Charlie', 'c@c.com', 28)
```
> **性能提示**：对于 MySQL 数据库，确保你的数据库连接 URL 中包含了 `rewriteBatchedStatements=true` 参数，这样才能真正开启 JDBC 驱动层面的批量处理，获得巨大的性能提升。

---

### 高级技巧与注意事项

1. **处理空集合**：如果传入的 `collection` 为 `null` 或者为空列表，`<foreach>` 标签不会生成任何内容。这在大多数情况下是安全的。但有时可能会导致语法错误，比如 `WHERE id IN` 后面什么都没有。为了避免这种情况，可以加一个 `<if>` 判断：

 ```xml
 <select id="findUsersByIds" resultType="com.example.User">
 SELECT * FROM user
 <if test="userIds != null and userIds.size() > 0">
 WHERE id IN
 <foreach collection="userIds" item="userId" open="(" separator="," close=")">
 #{userId}
 </foreach>
 </if>
 </select>
 ```

2. **迭代 Map**：当 `collection` 是一个 Map 时，`index` 会成为 Map 的 `key`，而 `item` 会成为 Map 的 `value`。

 ```xml
 <foreach collection="myMap" index="key" item="value" separator=",">
 #{key} = #{value}
 </foreach>
 ```

### 总结

`<foreach>` 是 MyBatis 动态 SQL 的瑞士军刀。它通过简单的属性配置，优雅地解决了处理集合参数时需要动态拼接 SQL 的复杂问题。熟练掌握 `IN` 子句和批量插入这两个核心用法，能够极大地提升开发效率和程序性能。记住，始终优先使用 `@Param` 为你的集合参数命名，这是一种让代码更清晰、更稳健的良好习惯。
