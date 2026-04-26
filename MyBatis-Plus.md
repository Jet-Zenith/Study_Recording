# MyBatis-Plus 笔记

MyBatis-Plus (MP) 的核心理念是“只做增强不做改变”。在保持 MyBatis 原有灵活性的基础上，MP 通过高度抽象的 API 和插件机制，极大提升了单表 CRUD 操作的开发效率。

## 1. 核心架构与起步配置

### 1.1 依赖引入
在 Spring Boot 工程中，直接引入官方提供的 starter 即可实现数据源与 MyBatis 核心组件的自动装配。

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3.1</version>
</dependency>
```

### 1.2 Mapper 接口层

数据访问层接口仅需继承 `BaseMapper<T>`，底层由 MP 通过动态代理在启动时注入基础的 SQL 语句。

```Java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 基础的 insert/update/delete/selectById 已自动实现
}
```

------

## 2. 领域模型映射与自动填充

### 2.1 核心注解体系

MP 提供了一套完整的注解用于建立实体类（Entity）与数据库表的映射关系：

- **`@TableName("table_name")`**：显式指定映射的物理表名。
- **`@TableId`**：指定主键。其 `type` 属性决定了主键生成策略，企业级系统常使用 `IdType.ASSIGN_ID`（雪花算法生成分布式唯一 ID）。
- **`@TableField`**：解决字段名不一致、关键字冲突或标识非数据库字段（`exist = false`）。

### 2.2 审计字段自动填充 (MetaObjectHandler)

在标准的工程规范中，表中通常会包含 `create_time` 和 `update_time` 等审计字段。MP 提供了 `MetaObjectHandler` 接口实现这些字段的自动赋值，避免在业务代码中冗余处理。

**1. 实体类配置：**

```Java
@Data
@TableName("t_user")
public class User {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
    
    private String username;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
}
```

**2. 填充器实现：**

```Java
@Component
public class MybatisMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

------

## 3. 条件构造与链式编程

### 3.1 LambdaWrapper 体系

为了避免硬编码列名导致的运行时错误，强烈建议在业务层一律使用 `LambdaQueryWrapper` 和 `LambdaUpdateWrapper`，通过方法引用（Method Reference）获取字段名。

```Java
// 传统的 QueryWrapper (易因拼写错误导致异常)
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("user_name", "admin").ge("age", 18);

// 推荐的 LambdaQueryWrapper (类型安全)
LambdaQueryWrapper<User> lambdaWrapper = new LambdaQueryWrapper<>();
lambdaWrapper.eq(User::getUsername, "admin")
             .ge(User::getAge, 18);
List<User> users = userMapper.selectList(lambdaWrapper);
```

### 3.2 Service 层链式调用 (ChainWrapper)

MP 提供了 `IService` 接口的实现类 `ServiceImpl`，在 Controller 或 Facade 层中，可以使用更加优雅的链式编程 API：

```Java
// 使用链式查询获取第一条符合条件的数据
User activeUser = userService.lambdaQuery()
        .eq(User::getStatus, 1)
        .likeRight(User::getUsername, "Jack") // 匹配 Jack%
        .orderByDesc(User::getCreateTime)
        .last("LIMIT 1")
        .one();

// 使用链式更新局部字段
boolean updateResult = userService.lambdaUpdate()
        .set(User::getBalance, new BigDecimal("1000.00"))
        .eq(User::getId, 1001L)
        .update();
```

------

## 4. 高级工程特性

### 4.1 逻辑删除机制

配置全局逻辑删除后，底层的 CRUD 方法会自动加上隐藏条件。

**配置 (`application.yml`)：**

```YAML
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
```

执行 `userMapper.deleteById(1L)` 时，实际执行的 SQL 变更为：`UPDATE t_user SET deleted=1 WHERE id=1 AND deleted=0`。

### 4.2 类型处理器 (TypeHandler)

针对数据库中存储的 JSON 字符串，MP 提供了原生支持，能够自动实现与 Java 对象/集合的序列化与反序列化。

```Java
@Data
@TableName(value = "t_order", autoResultMap = true) // 必须开启 autoResultMap
public class Order {
    private Long id;
    
    // 将该字段的 JSON 数据自动映射为 Java 的 Map 集合
    @TableField(typeHandler = JacksonTypeHandler.class)
    private Map<String, Object> extraInfo;
}
```

------

## 5. 插件生态体系 (MybatisPlusInterceptor)

MP 基于 MyBatis 的拦截器机制实现了多种核心插件，需要在配置类中统一注册。

### 5.1 统一拦截器配置

```Java
@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        
        // 1. 添加乐观锁插件
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        
        // 2. 添加分页插件 (注意：必须放在最后)
        PaginationInnerInterceptor pageInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
        pageInterceptor.setMaxLimit(1000L); // 限制单页最大请求数，防止内存溢出
        interceptor.addInnerInterceptor(pageInterceptor);
        
        return interceptor;
    }
}
```

### 5.2 并发控制：乐观锁实现

在应对并发更新（如扣减余额、库存）时，乐观锁是一种轻量级的解决方案。

1. 在实体类对应字段加上 `@Version` 注解。

```Java
@Version
private Integer version;
```

1. 更新操作必须先查询出数据，带上 version 版本号进行更新。

```Java
// 1. 先查询，获取当前 version
User user = userMapper.selectById(1L);

// 2. 修改业务数据
user.setBalance(user.getBalance().subtract(new BigDecimal("100")));

// 3. 执行更新 (MP 底层会自动追加 WHERE id = 1 AND version = 旧版本号)
int rows = userMapper.updateById(user);
if (rows == 0) {
    throw new RuntimeException("数据已被修改，请重试"); // 工程中建议抛出具体的自定义领域异常
}
```

### 5.3 分页查询实践

```Java
public Page<User> getUsersByCondition(int pageNo, int pageSize, String username) {
    Page<User> pageInfo = new Page<>(pageNo, pageSize);
    
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    queryWrapper.like(StringUtils.isNotBlank(username), User::getUsername, username)
                .orderByDesc(User::getCreateTime);
                
    // 将分页对象和查询条件传入
    return userMapper.selectPage(pageInfo, queryWrapper);
}
```

------

## 6. 自定义 SQL 与复杂联合查询

当面临复杂的多表 Join 查询时，建议回归 MyBatis 的原生写法（XML 或注解），但仍然可以复用 MP 的 `Wrapper` 来动态生成 Where 条件。

**Mapper 接口：**

```Java
// 使用 @Param("ew") 注入 Wrapper 对象
@Select("SELECT u.*, r.role_name FROM t_user u " +
        "LEFT JOIN t_role r ON u.role_id = r.id " +
        "${ew.customSqlSegment}")
List<UserRoleDTO> selectUserWithRole(@Param(Constants.WRAPPER) Wrapper<User> wrapper);
```

**业务调用：**

```Java
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getStatus, 1).gt(User::getAge, 20);

// MP 会自动将 Wrapper 解析为 SQL 片段并拼接到自定义 SQL 尾部
List<UserRoleDTO> result = userMapper.selectUserWithRole(wrapper);
```