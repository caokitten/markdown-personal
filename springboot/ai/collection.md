## 🤖 Assistant

好的，我们来深入探讨一下 `<foreach>` 标签中 `collection` 属性的工作原理。这是 MyBatis 的一个基础概念，彻底理解它能帮你避免很多常见错误。

### 核心概念：你数据的“地址”

你可以把 `collection` 属性想象成一个**“地址”**，它告诉 MyBatis 在你传递给 Mapper 方法的众多参数中，究竟去哪里找到那个可迭代的数据（比如 `List`, `Set`, `Array` 或 `Map`）。

当你调用一个 Mapper 方法时，MyBatis 并不会直接把参数传递给 SQL。它会先将这些参数打包成一个内部的**参数Map**（一个键值对结构）。`collection` 属性的值，就是 MyBatis 在这个内部 Map 中查找你的数据时所使用的**键（key）**。

这个参数 Map 是如何被填充的，完全取决于你如何定义你的 Java Mapper 方法签名。这正是“工作原理”的根源所在。

---

### 搜索机制：MyBatis 如何找到你的集合

MyBatis 遵循一套清晰的规则来查找 `collection` 属性指定的集合。让我们逐一分析不同的场景。

#### 情况一：使用 `@Param` 注解（最明确、最推荐的方式）

这是最直接、最可靠的方法。你通过 `@Param` 注解为你的参数显式地命名。

**1. Mapper 接口 (Java):**
```java
public interface UserMapper {
    // 我们明确地将 List 参数命名为 "userIds"
    List<User> findUsersByIds(@Param("userIds") List<Integer> ids, @Param("status") String status);
}
```

**2. Mapper XML:**
```xml
<select id="findUsersByIds" resultType="com.example.User">
    SELECT * FROM user
    WHERE status = #{status} AND id IN
    <foreach collection="userIds" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

**工作原理：**
1. 当你调用 `findUsersByIds(...)` 时，MyBatis 创建其内部的参数 Map。
2. 因为有 `@Param("userIds")`，它会将你的 `List<Integer>` 以 `"userIds"` 为键，放入这个 Map 中。
3. 同时，它也会将 status 字符串以 `"status"` 为键放入 Map。
4. `<foreach>` 标签看到 `collection="userIds"`。它在参数 Map 中查找键 `"userIds"`，找到了列表，然后开始对其进行迭代。

> **结论：** 使用 `@Param` 为你的数据提供了一个清晰、无歧义的“地址”。这是最佳实践，尤其是在你有多个参数的时候。

---

#### 情况二：单个列表/数组/Set参数（不使用 `@Param`）

如果你的 Mapper 方法**只有一个参数**，并且该参数是 `List`、`Array` 或 `Set`，MyBatis 会使用一套默认的命名约定。

**1. Mapper 接口 (Java):**
```java
public interface UserMapper {
    // 只有一个参数，是一个 List
    List<User> findUsersByIds(List<Integer> ids);
}
```

**2. Mapper XML:**
```xml
<select id="findUsersByIds" resultType="com.example.User">
    SELECT * FROM user
    WHERE id IN
    <!-- 在这里你可以使用 "list" 或 "collection" -->
    <foreach collection="list" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

**工作原理：**
1. MyBatis 看到一个 `List` 类型的单一参数。
2. 它会使用一个**默认的键**将这个列表放入参数 Map 中。这些默认键是：
 * 对于 `List`: `"list"` (为了兼容性，也接受 `"collection"`)。
 * 对于 `Array`: `"array"`。
 * 对于 `Set`: `"collection"` (在某些版本中也可能是 `"set"`，但 `"collection"` 更通用)。
3. 带有 `collection="list"` 的 `<foreach>` 标签在参数 Map 中查找键 `"list"`，找到数据，然后进行迭代。

> **警告：** 这种方式只适用于单个参数。如果你添加了第二个参数，但没有对所有参数都使用 `@Param`，MyBatis 就会混淆，并抛出类似 `Parameter 'list' not found` 的错误。

---

#### 情况三：参数是POJO（简单Java对象）/Bean

你可以传递一个单独的对象，该对象的一个属性是你的集合。

**1. POJO 类 (Java):**
```java
public class SearchCriteria {
    private List<Integer> ids;
    private String status;
    // ... getters and setters
}
```

**2. Mapper 接口 (Java):**
```java
public interface UserMapper {
    // 单个参数是我们的 POJO 对象
    List<User> findUsersByCriteria(SearchCriteria criteria);
}
```

**3. Mapper XML:**
```xml
<select id="findUsersByCriteria" resultType="com.example.User">
    SELECT * FROM user
    WHERE status = #{status} AND id IN
    <!-- 使用点标记法来访问属性 -->
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```
等一下，为什么是 `"ids"` 而不是 `"criteria.ids"`？

**工作原理：**
1. MyBatis 接收到 `SearchCriteria` 对象。默认情况下，它会认为该对象的**属性可以直接在参数 Map 的上下文中使用**。
2. 它实际上“解包”了 `criteria` 对象的属性。所以它可以直接通过 `#{status}` 找到 `status` 属性，直接通过 `collection="ids"` 找到 `ids` 列表。
3. `<foreach>` 标签查找一个名为 `"ids"` 的键，它会在你的 `SearchCriteria` 对象上找到 `getIds()` 方法，获取列表，然后进行迭代。

*注意：在某些边缘情况或为了更清晰，你可能会写成 `collection="criteria.ids"`，前提是你给参数加了注解 `@Param("criteria") SearchCriteria criteria`。但当 Bean 是唯一参数时，其属性通常是直接可访问的。*

---

#### 情况四：参数是 `Map`

你也可以直接传递一个 `Map`。`collection` 属性的值就是列表中那个 `List` 在 Map 中的键。

**1. Mapper 接口 (Java):**
```java
public interface UserMapper {
    List<User> findUsersByMap(Map<String, Object> params);
}
```

**2. Service 层代码 (如何调用):**
```java
Map<String, Object> myParams = new HashMap<>();
myParams.put("status", "active");
myParams.put("userIds", Arrays.asList(1, 2, 3)); // 我们的列表存放在 "userIds" 这个键下
userMapper.findUsersByMap(myParams);
```

**3. Mapper XML:**
```xml
<select id="findUsersByMap" resultType="com.example.User">
    SELECT * FROM user
    WHERE status = #{status} AND id IN
    <foreach collection="userIds" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

**工作原理：**
1. `Map` 对象 `myParams` 被传入。
2. MyBatis 将这个 Map 的键（`"status"`, `"userIds"`）视为其内部参数 Map 的键。
3. `<foreach>` 标签的 `collection="userIds"` 完美地匹配了 Map 中存储列表的那个键。

### 解析逻辑总结

MyBatis 大致按照以下顺序来解析 `collection` 属性，并在其内部参数 Map 中查找匹配的键：

1. **与 `@Param` 名称直接匹配：** 这是最直接的查找。如果你有 `@Param("myList")`，它就会查找 `"myList"`。
2. **Bean 参数的属性：** 如果参数是一个 Bean，它会检查该 Bean上是否有同名的属性（例如，`collection="ids"` 会查找 `getIds()` 方法）。
3. **Map 参数的键：** 如果参数是一个 Map，它会在 Map 中查找同名的键。
4. **默认名称：** 如果是兼容类型的单个参数，并且没有其他规则匹配上，它会检查默认名称（`"list"`, `"collection"`, `"array"`）。

理解这个层级关系就解释了为什么使用 `@Param` 是最稳健的策略——它消除了所有歧义，并且不依赖于默认行为。
