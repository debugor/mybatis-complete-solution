<a name="PmRJm"></a>
## 4.1 介绍
MyBatis 动态 SQL 是一种强大的特性，允许你根据不同的条件来构建动态的 SQL 查询语句，以适应不同的业务需求。动态 SQL 可以帮助你在映射文件中编写更加灵活、可维护的查询语句，而不需要通过拼接字符串来实现条件的动态变化。

MyBatis 支持多种动态 SQL 元素，以下是其中一些常见的元素：

1. `**<if>**`** 元素：** 用于条件判断，根据条件来包含或排除 SQL 片段。

```xml
<select id="getUserByUsernameAndPassword" resultType="User">
    SELECT * FROM users
    WHERE username = #{username}
    <if test="password != null">
        AND password = #{password}
    </if>
</select>
```

2. `**<choose>**`**、**`**<when>**`**、**`**<otherwise>**`** 元素：** 类似于 Java 中的 `switch` 语句，根据不同条件选择不同的 SQL 片段。

```xml
<select id="getUsersByCondition" resultType="User" parameterType="map">
    SELECT * FROM users
    <where>
        <choose>
            <when test="username != null">
                AND username = #{username}
            </when>
            <when test="email != null">
                AND email = #{email}
            </when>
            <otherwise>
                AND status = 'active'
            </otherwise>
        </choose>
    </where>
</select>
```

3. `**<trim>**`** 元素：** 可以用于在 SQL 语句的开始或结束去除多余的空格和逗号，使生成的 SQL 更加规整。

```xml
<select id="getUsersByCondition" resultType="User" parameterType="map">
    SELECT * FROM users
    <where>
        <trim prefix="AND" prefixOverrides="OR">
            <if test="username != null">
                OR username = #{username}
            </if>
            <if test="email != null">
                OR email = #{email}
            </if>
        </trim>
    </where>
</select>
```

4. `**<foreach>**`** 元素：** 用于遍历集合或数组，生成对应的 SQL 片段。常用于 `IN` 子句。

```xml
<select id="getUsersByIds" resultType="User">
    SELECT * FROM users
    WHERE id IN
    <foreach collection="userIds" item="userId" open="(" separator="," close=")">
        #{userId}
    </foreach>
</select>
```

这些动态 SQL 元素使你能够在映射文件中编写更加灵活和可读的查询语句，根据不同的条件生成不同的 SQL 片段，从而避免了繁琐的拼接字符串操作。通过合理利用动态 SQL，你可以轻松实现复杂的查询逻辑和灵活的查询条件，提高代码的可维护性和可扩展性。


<a name="SffE8"></a>
## 4.2 真实场景介绍
当使用 MyBatis 动态 SQL 时，可以根据不同的实际场景来灵活地构建查询语句。以下是几个实际场景，以及如何在映射文件中使用动态 SQL 来实现。

**场景一：根据条件查询用户列表**

假设有一个需求，根据不同的条件查询用户列表。条件包括用户名、邮箱和状态。如果某些条件为空，应忽略这些条件。

```xml
<select id="getUsersByCondition" resultType="User" parameterType="map">
    SELECT * FROM users
    <where>
        <if test="username != null">
            AND username = #{username}
        </if>
        <if test="email != null">
            AND email = #{email}
        </if>
        <if test="status != null">
            AND status = #{status}
        </if>
    </where>
</select>
```

在这个示例中，通过使用 `<if>` 元素，可以根据不同的条件动态地生成不同的查询语句。如果某个条件为空，对应的查询条件将被忽略。

**场景二：根据多个 ID 查询用户列表**

假设需要根据多个用户 ID 查询用户列表，可以使用 `<foreach>` 元素来遍历 ID 列表。

```xml
<select id="getUsersByIds" resultType="User">
    SELECT * FROM users
    WHERE id IN
    <foreach collection="userIds" item="userId" open="(" separator="," close=")">
        #{userId}
    </foreach>
</select>
```

在这个示例中，通过使用 `<foreach>` 元素，可以将传入的用户 ID 列表转换为逗号分隔的 ID 列表，从而生成查询条件。

**场景三：根据动态排序查询用户列表**

假设需要根据不同的排序条件查询用户列表，可以使用 `<choose>` 和 `<when>` 元素来实现动态排序。

```xml
<select id="getUsersByOrder" resultType="User" parameterType="map">
    SELECT * FROM users
    <where>
        <choose>
            <when test="orderBy == 'username'">
                ORDER BY username
            </when>
            <when test="orderBy == 'email'">
                ORDER BY email
            </when>
            <otherwise>
                ORDER BY id
            </otherwise>
        </choose>
    </where>
</select>
```

在这个示例中，通过使用 `<choose>` 和 `<when>` 元素，根据传入的排序条件动态生成不同的排序语句。

这些实际场景示例演示了如何使用动态 SQL 元素来根据不同的条件构建查询语句。动态 SQL 使查询语句更加灵活、可维护，并且能够适应不同的业务需求。根据具体的查询逻辑和业务需求，你可以进一步组合和定制动态 SQL 元素，以满足更复杂的查询场景。
