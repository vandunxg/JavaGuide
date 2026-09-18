---
title: "Tổng hợp câu hỏi phỏng vấn MyBatis thường gặp"
description: "Giải thích chi tiết câu hỏi phỏng vấn MyBatis thường gặp, bao quát execution flow, Mapper dynamic proxy, #{}, ${}, dynamic SQL, cache cấp một, cache cấp hai, pagination plugin, Executor, batch processing và Spring transaction."
category: Framework
icon: "mdi:database-outline"
tag:
  - MyBatis
head:
  - - meta
    - name: keywords
      content: "MyBatis interview questions, MyBatis execution flow, Mapper dynamic proxy, #{} và ${}, dynamic SQL, cache cấp một, cache cấp hai, pagination plugin, BatchExecutor, MyBatis plugin"
---

> Bài viết này ban đầu được tổng hợp từ tài liệu trên Internet, không thể xác nhận nguồn gốc ban đầu. Lần viết lại này chủ yếu dựa trên tài liệu chính thức của MyBatis 3.5.x và MyBatis-Spring, đồng thời bổ sung các câu hỏi thường gặp về cache, batch processing, Spring transaction và xử lý result set lớn.

## Kiến thức cơ bản về MyBatis

### MyBatis là gì? Tại sao nói đây là ORM bán tự động?

MyBatis là một framework persistence layer. Nó đóng gói các công việc lặp lại trong JDBC như tạo connection, thiết lập parameter, thực thi SQL, duyệt result set và đóng resource, đồng thời cho phép developer tự viết SQL và mapping parameter cùng query result thành object Java.

MyBatis thường được gọi là ORM bán tự động vì SQL, field mapping và associated query thường vẫn do developer kiểm soát. Các ORM như Hibernate/JPA nhấn mạnh việc generate SQL dựa trên entity relationship và mapping metadata, còn MyBatis trao quyền kiểm soát SQL cho developer.

Thiết kế này phù hợp với các project có SQL phức tạp, cần tối ưu chính xác hoặc sử dụng nhiều tính năng database; đổi lại, SQL và mapping code nhiều hơn, việc migrate database dialect, associated loading và batch operation cũng cần developer tự xử lý.

### MyBatis khác JPA/Hibernate như thế nào?

| Hạng mục so sánh   | MyBatis                                                 | JPA/Hibernate                                               |
| ------------------ | ------------------------------------------------------- | ----------------------------------------------------------- |
| SQL                | Thường do developer viết                                | Thường do framework generate dựa trên mapping               |
| Mức độ kiểm soát   | Dễ kiểm soát chính xác SQL, index và tính năng database | Tập trung hơn vào object model và persistence state         |
| Chi phí phát triển | Nhiều mapping và SQL hơn                                | Ít code CRUD thông thường hơn                               |
| Database migration | SQL viết tay có thể phụ thuộc dialect                   | Query tiêu chuẩn thường portable hơn                        |
| Associated query   | Chọn rõ Join hoặc nested query                          | Hỗ trợ object association và fetch strategy                 |
| Rủi ro thường gặp  | SQL phân tán, mapping sai, injection do nối chuỗi       | N+1, fetch phạm vi quá lớn, khó kiểm soát SQL được generate |

Không có giải pháp nào phù hợp với mọi project. Khi có nhiều CRUD đơn giản và domain model ổn định, JPA/Hibernate có thể giảm code lặp; khi có report, query phức tạp và nhiều SQL optimization, MyBatis trực tiếp hơn. Cũng có thể lựa chọn theo nhu cầu ở các module khác nhau trong cùng một system.

### MyBatis có những core component nào?

- **`SqlSessionFactoryBuilder`**: Đọc configuration và tạo `SqlSessionFactory`. Sau khi build xong, thông thường có thể giải phóng.
- **`SqlSessionFactory`**: Tạo `SqlSession`, chi phí build cao, thông thường giữ lại một instance trong application.
- **`SqlSession`**: Thực thi SQL, quản lý transaction và lấy Mapper. Nó không thread-safe, nên giới hạn trong một request, một method hoặc một transaction.
- **`Configuration`**: Lưu global setting, `MappedStatement`, `ResultMap`, `TypeHandler` và runtime metadata của plugin.
- **`MappedStatement`**: Tương ứng với một mapping `<select>`, `<insert>`, `<update>` hoặc `<delete>`.
- **`Executor`**: Chịu trách nhiệm query, update, cache cấp một và các lời gọi liên quan đến transaction.
- **`StatementHandler`**: Tạo và thao tác với JDBC `Statement`.
- **`ParameterHandler`**: Thiết lập parameter vào `PreparedStatement`.
- **`ResultSetHandler`**: Mapping JDBC `ResultSet` thành object Java.

`SqlSource` lưu logic generate SQL; khi thực thi thật, nó tạo `BoundSql` dựa trên parameter. `BoundSql` chứa SQL cuối cùng, parameter mapping và additional parameter.

### SqlSessionFactory, SqlSession và Mapper có thread-safe không?

Sau khi build xong, `SqlSessionFactory` có thể được share trong application. `SqlSession` chứa database connection, transaction và state thay đổi như cache cấp một, không nên share giữa các thread. Mapper proxy lấy từ một `SqlSession` cũng nên tuân theo lifecycle tương tự.

Trong MyBatis-Spring, Mapper được inject được `SqlSessionTemplate` hỗ trợ. `SqlSessionTemplate` thread-safe, nó tìm `SqlSession` tương ứng với Spring transaction hiện tại, đồng thời chịu trách nhiệm quản lý lifecycle như commit, rollback và close. Vì vậy Spring singleton Service có thể inject Mapper, nhưng không nên lưu native `SqlSession` được tạo thủ công vào field singleton.

### ⭐️Nguyên lý hoạt động của Mapper interface là gì?

MyBatis tạo proxy object cho Mapper interface thông qua JDK dynamic proxy. Khi gọi method của Mapper, flow đại khái như sau:

1. `MapperProxy` intercept method của interface.
2. MyBatis lấy Statement ID dựa trên fully qualified name và method name của Mapper interface, ví dụ `com.example.UserMapper.selectById`.
3. `MapperMethod` phân tích parameter và return type, gọi các method tương ứng của `SqlSession` như `selectOne()`, `selectList()`, `insert()`.
4. `SqlSession` tìm `MappedStatement` dựa trên Statement ID rồi giao cho `Executor` thực thi.
5. Query result được `ResultSetHandler` mapping, sau đó trả về theo return type khai báo trong Mapper method.

Bản thân Mapper interface thường không có implementation class. Fully qualified name của interface phải giống `namespace` trong XML, method name phải giống `id` của mapping statement.

### Method của Mapper interface có thể overload không?

Java cho phép khai báo method overload trong Mapper interface, nhưng khi tìm mapping statement, MyBatis sử dụng “fully qualified name của interface + method name”, không đưa parameter type vào Statement ID. Trong cùng `namespace` của XML cũng không thể định nghĩa hai mapping có cùng `id`.

Trong một số ít trường hợp, nhiều method overload có thể dùng chung dynamic SQL, nhưng MyBatis không thể chọn SQL theo parameter signature như Java compiler; parameter name, return type và dynamic condition cũng rất dễ xung đột. Trong project thực tế nên dùng method name khác nhau để thể hiện các query khác nhau.

### ⭐️Flow đầy đủ khi MyBatis thực thi một query là gì?

Có thể trả lời theo hai phần: “parse ở startup” và “execute ở runtime”.

Startup:

1. Đọc global configuration của MyBatis, tạo `Configuration`.
2. Parse Mapper XML hoặc annotation, đăng ký SQL, parameter mapping, result mapping thành `MappedStatement`, `ResultMap` và `SqlSource`.
3. Tạo `SqlSessionFactory`; trong project Spring, tiếp tục đăng ký Mapper proxy.

Runtime:

1. Gọi Mapper proxy, tìm `MappedStatement` dựa trên interface name và method name.
2. `SqlSource` tạo `BoundSql` dựa trên argument thực tế.
3. `Executor` kiểm tra cache cấp một theo rule trước; nếu cache miss thì tiếp tục truy cập database.
4. `StatementHandler` tạo JDBC Statement, `ParameterHandler` thiết lập parameter.
5. JDBC thực thi SQL, `ResultSetHandler` mapping result thành object.
6. Result được ghi vào cache theo rule rồi trả về; sau khi transaction kết thúc thì commit hoặc rollback.

Plugin có thể wrap `Executor`, `StatementHandler`, `ParameterHandler` và `ResultSetHandler`, vì vậy pagination, audit và SQL rewrite có thể được chèn vào flow này.

## Xử lý parameter và dynamic SQL

### ⭐️`#{}` và `${}` khác nhau như thế nào?

Trong mapping SQL:

- `#{}` tạo JDBC parameter placeholder `?`, sau đó MyBatis thiết lập parameter thông qua `ParameterHandler` và `TypeHandler`. Parameter được truyền tới database dưới dạng data, thường có thể ngăn SQL injection tại vị trí đó.
- `${}` thay trực tiếp kết quả expression vào SQL text, không tạo bound parameter. Nó có thể thay đổi column name, table name, sort direction hoặc cả đoạn SQL, do đó cũng có rủi ro injection.

```xml
<select id="findByName" resultType="User">
  SELECT id, name
  FROM users
  WHERE name = #{name}
  ORDER BY ${orderBy}
</select>
```

`name` nên dùng `#{}`. Nếu `orderBy` đến trực tiếp từ request, attacker có thể thay đổi SQL; nên mapping external value thành column name và sort direction được định nghĩa sẵn ở server trong Java code.

`${}` cũng xuất hiện trong MyBatis configuration file, ví dụ `${driver}`, `${url}`. Cách viết này là configuration property substitution, thường hoàn tất khi parse configuration, khác với scenario dynamic text substitution trong mapping SQL.

### Khi Mapper method có nhiều parameter, tham chiếu parameter như thế nào?

Khuyến nghị dùng `@Param` để chỉ rõ parameter name:

```java
User selectByTenantAndId(
    @Param("tenantId") Long tenantId,
    @Param("userId") Long userId
);
```

```xml
<select id="selectByTenantAndId" resultType="User">
  SELECT id, name
  FROM users
  WHERE tenant_id = #{tenantId}
    AND id = #{userId}
</select>
```

Khi không có `@Param`, MyBatis cung cấp các tên chung như `param1`, `param2`; nếu project compile với `-parameters` và bật `useActualParamName`, cũng có thể đọc parameter name thực tế. Tuy nhiên thay đổi compile configuration có thể khiến parameter name thực tế không còn dùng được, nên dùng `@Param` trong public Mapper sẽ ổn định hơn.

Collection parameter có thể được truy cập bằng các name như `collection`, `list` hoặc `array`; name cụ thể phụ thuộc vào parameter type và `@Param`. Gắn rõ `@Param("ids")` cho collection rồi dùng trong `<foreach collection="ids">` sẽ dễ đọc hơn.

### ⭐️MyBatis dynamic SQL có những tag nào? Nguyên lý là gì?

Các tag thường dùng gồm:

- `<if>`: Ghép SQL theo condition.
- `<choose>`, `<when>`, `<otherwise>`: Chọn một branch trong nhiều branch.
- `<trim>`, `<where>`, `<set>`: Xử lý prefix, suffix, `AND`, comma dư thừa và các vấn đề tương tự.
- `<foreach>`: Duyệt collection, thường dùng cho condition `IN` và batch insert.
- `<bind>`: Tạo variable có thể dùng trong dynamic SQL hiện tại.

Khi parse dynamic SQL, MyBatis xây dựng các XML node thành một cấu trúc kết hợp `SqlNode`. Sau khi thực thi Mapper method, mỗi node quyết định có output SQL hay không dựa trên parameter context và OGNL expression, cuối cùng `DynamicSqlSource` tạo `BoundSql`.

```xml
<select id="find" resultType="User">
  SELECT id, name, status
  FROM users
  <where>
    <if test="name != null and name != ''">
      name = #{name}
    </if>
    <if test="status != null">
      AND status = #{status}
    </if>
  </where>
</select>
```

`<where>` chỉ output `WHERE` khi bên trong có content, đồng thời xử lý `AND` hoặc `OR` dư thừa ở đầu.

### Mapper XML có những element thường dùng nào?

- `<cache>`, `<cache-ref>`: Cấu hình cache cấp hai của namespace hiện tại hoặc tham chiếu cache của namespace khác.
- `<resultMap>`: Định nghĩa mapping giữa result set và object.
- `<sql>`, `<include>`: Định nghĩa và reuse SQL fragment.
- `<select>`, `<insert>`, `<update>`, `<delete>`: Định nghĩa mapping statement.
- `<selectKey>`: Thực thi query lấy primary key trước hoặc sau insert.

Tài liệu cũ thường nhắc đến `<parameterMap>`, nhưng element này đã deprecated, nên dùng inline parameter mapping.

`<sql>` có thể được định nghĩa sau `<include>` tham chiếu đến nó. Khi parse Mapper XML, MyBatis sẽ collect và register các `<sql>` fragment trong file hiện tại trước, rồi parse các statement CRUD, vì vậy fragment được tham chiếu không bắt buộc phải viết trước. Với reference giữa các resource chưa thể resolve tạm thời, MyBatis còn đưa chúng vào pending set và retry sau khi definition liên quan được load. Tuy vậy, mapping file vẫn nên được tổ chức theo thứ tự được khuyến nghị trong tài liệu chính thức để tránh circular reference và dependency cross-file khó bảo trì.

### Mapper XML được parse thành những internal object nào?

- Mỗi `<select>`, `<insert>`, `<update>`, `<delete>` được register thành một `MappedStatement`.
- Static SQL thường tương ứng với `RawSqlSource`, dynamic SQL thường tương ứng với `DynamicSqlSource`.
- Khi execute, `SqlSource` tạo `BoundSql`, trong đó chứa SQL cuối cùng và danh sách `ParameterMapping`.
- `<resultMap>` được parse thành `ResultMap`, nested mapping được parse thành `ResultMapping`.
- `<cache>` tạo và decorate một cache instance cấp namespace.

Các object này cuối cùng được lưu trong `Configuration`. Trong thời gian application chạy, thông thường chỉ đọc các metadata này, không nên tùy tiện thay đổi global `Configuration`.

## Result mapping

### `resultType` và `resultMap` khác nhau như thế nào?

`resultType` chỉ định trực tiếp type của object trả về, phù hợp với query đơn giản có column name dễ tự động match với property name. SQL column alias cũng có thể hỗ trợ match, ví dụ `user_name AS userName`.

`resultMap` mô tả rõ quan hệ giữa column và property, đồng thời hỗ trợ constructor mapping, type handler, discriminator và nested `association`, `collection`, phù hợp với object phức tạp và associated query.

```xml
<resultMap id="userMap" type="User">
  <id property="id" column="user_id" />
  <result property="userName" column="user_name" />
</resultMap>
```

Automatic mapping có thể kết hợp với `mapUnderscoreToCamelCase=true` để mapping `user_name` thành `userName`. Với query Join phức tạp không nên quá phụ thuộc vào automatic mapping; column trùng name nên dùng alias và cấu hình rõ trong `resultMap`.

### ⭐️`association` và `collection` khác nhau như thế nào?

- `<association>` mapping quan hệ “có một”, ví dụ order tương ứng với một user.
- `<collection>` mapping quan hệ “có nhiều”, ví dụ user tương ứng với nhiều order.

Có hai cách associated loading thường gặp:

1. **Nested result mapping (Nested Results)**: Dùng Join để query object chính và associated object trong một lần, sau đó merge các row trùng dựa trên `<id>` và mapping khác. Số lần SQL ít, nhưng result set có thể phình to.
2. **Nested query (Nested Select)**: Query object chính trước, sau đó thực thi một mapping statement khác cho associated property. Cấu trúc đơn giản, cũng hỗ trợ lazy loading, nhưng dễ tạo N+1 query.

Khi dùng Join để mapping one-to-many, object chính và object con đều nên cấu hình đúng `<id>`. MyBatis sẽ dựa trên các identifier này để reuse object đã có và lắp ráp collection; thiếu identifier có thể làm tăng chi phí tạo object và mapping, thậm chí cho kết quả sai.

### N+1 query là gì? Làm thế nào để tránh?

Query N main record trước, sau đó query associated object một lần cho từng record, tổng cộng thực thi 1 + N SQL, đó là N+1 query thường được nhắc đến. Nested query và lazy loading đều có thể trigger vấn đề này.

Các cách xử lý thường gặp:

- Dùng Join và nested result mapping để query một lần.
- Batch query main record trước, sau đó batch query associated record theo primary key collection, cuối cùng lắp ráp trong memory.
- Trong các scenario như GraphQL/DataLoader, batch merge các request cùng một batch.
- Chỉ dùng lazy loading khi thực sự sẽ truy cập ít associated property, đồng thời xác nhận số lần query bằng SQL monitoring.

Join cũng không phải càng nhiều càng tốt. Khi one-to-many có nhiều tầng, Cartesian product sẽ khiến result set tăng đột biến; batch query thành hai lần thường dễ kiểm soát hơn.

### MyBatis có hỗ trợ lazy loading không? Nguyên lý là gì?

MyBatis có thể lazy load `association` và `collection` được cấu hình bằng nested query. Sau khi bật `lazyLoadingEnabled`, MyBatis dùng proxy object để lưu property chờ load; khi truy cập property liên quan, proxy sẽ thực thi query đã đăng ký rồi ghi result trở lại.

`fetchType="lazy"` hoặc `fetchType="eager"` có thể override global setting của từng association. MyBatis 3.5.x mặc định dùng Javassist để tạo proxy lazy loading; CGLIB đã deprecated từ 3.5.10.

Lazy loading có hai vấn đề thường gặp: một là truy cập collection gây N+1; hai là chỉ truy cập property sau khi object đã rời khỏi phạm vi của `SqlSession` hoặc transaction, khiến việc load có thể không hoạt động. Khi interface trả về DTO, thường phù hợp hơn nếu query rõ toàn bộ data cần thiết trong Service.

### TypeHandler có tác dụng gì? Enum được mapping như thế nào?

`TypeHandler` chịu trách nhiệm chuyển đổi giữa Java type và JDBC type: khi thiết lập parameter cho `PreparedStatement`, chuyển Java value thành JDBC value; khi đọc `ResultSet`, chuyển ngược lại thành Java value.

MyBatis mặc định dùng `EnumTypeHandler` để lưu theo enum name, ví dụ `ACTIVE`. `EnumOrdinalTypeHandler` lưu theo ordinal của enum, nhưng dễ gây lỗi sau khi thay đổi thứ tự enum member, vì vậy thường không khuyến nghị dùng ordinal làm database value lâu dài.

Business enum thường có field `code` ổn định, có thể custom `BaseTypeHandler` để hoàn tất mapping giữa `code` và enum object. Khi register, cần chỉ rõ `javaType`, `jdbcType` nếu cần và rule xử lý null value.

### Query result rỗng hoặc mapping không đầy đủ thì nên kiểm tra thế nào?

Có thể kiểm tra theo thứ tự sau:

1. SQL có thực sự trả về data không, parameter value và JDBC type có đúng không.
2. `namespace`, Statement ID và Mapper method có tương ứng không.
3. Column alias có giống Java property không, có bật camel-case mapping như dự kiến không.
4. `column`, `property`, `javaType` và `typeHandler` của `resultMap` có đúng không.
5. Join có xuất hiện column trùng name bị ghi đè không, `<id>` của nested result có đầy đủ không.
6. Field có thiếu Setter, constructor parameter không match, hoặc method do Lombok generate có khác dự kiến không.
7. Cache cấp một có trả về query result trước đó trong cùng `SqlSession` không.

Khi mở MyBatis SQL log cũng cần quan sát đồng thời parameter cuối cùng; SQL chỉ có placeholder không thể cho biết query condition thực tế.

### `selectOne()` query ra nhiều record sẽ thế nào?

Khi không có record, `selectOne()` trả về `null`; khi đúng một record, trả về object đó; khi nhiều hơn một record, ném `TooManyResultsException`. Nó không tự động lấy record đầu tiên.

Nếu business yêu cầu result duy nhất, nên dùng unique constraint để đảm bảo data consistency. Chỉ dùng sort và `LIMIT 1` khi business xác định rõ cho phép “chọn bất kỳ một record” hoặc “lấy record mới nhất”, không được dùng nó để che giấu dirty data.

## MyBatis cache

### ⭐️Cache cấp một của MyBatis là gì?

Cache cấp một là local cache ở cấp `SqlSession`, scope mặc định là `SESSION`. Trong cùng một `SqlSession`, khi Statement, SQL cuối cùng, parameter, pagination, environment và các thông tin khác của hai query tạo thành cache key giống nhau, query lần hai có thể trả về trực tiếp cache result.

Cache cấp một cũng được dùng để xử lý circular reference trong nested result mapping. Khi đặt `localCacheScope` thành `STATEMENT`, local cache chỉ được dùng trong thời gian thực thi một statement, không còn share giữa hai query.

Các thao tác sau sẽ làm cache cấp một liên quan bị invalidate hoặc clear:

- Thực thi `insert`, `update`, `delete`.
- Gọi `clearCache()`.
- Commit, rollback hoặc close `SqlSession`.
- Query statement cấu hình `flushCache="true"`.

### Tại sao trong project Spring có cảm giác cache cấp một không hoạt động?

Cache cấp một đi theo `SqlSession` thực tế. Trong Spring, `SqlSessionTemplate` sẽ reuse `SqlSession` được bind với transaction hiện tại; nếu không có transaction, sau khi mỗi Mapper call hoàn tất, Session được tạo cho lần đó sẽ bị close, lần call tiếp theo có thể đã là một Session mới.

Vì vậy, khi gọi Mapper liên tiếp hai lần trong cùng một Service method nhưng không đi vào Spring transaction, không thể giả định rằng chúng chắc chắn share cache cấp một. Cũng không nên tùy tiện mở rộng transaction scope chỉ để “cache hit”; transaction length vẫn nên do business consistency quyết định.

### ⭐️Cache cấp hai của MyBatis là gì? Bật như thế nào?

Cache cấp hai được bind với `namespace` của Mapper XML và có thể được share bởi các `SqlSession` khác nhau. Global `cacheEnabled` mặc định được bật, nhưng từng namespace vẫn cần cấu hình `<cache/>`, hoặc thông qua `<cache-ref>` để tham chiếu cache của namespace khác.

```xml
<mapper namespace="com.example.UserMapper">
  <cache />

  <select id="selectById" resultType="User" useCache="true">
    SELECT id, name FROM users WHERE id = #{id}
  </select>
</mapper>
```

Query result thường phải chờ đến khi transaction commit mới visible với cache cấp hai của Session khác. Khi namespace đó thực thi insert, update hoặc delete, cache mặc định sẽ được clear. `useCache` và `flushCache` có thể điều chỉnh behavior trên từng statement.

Cấu hình cache mặc định có thể yêu cầu result object hỗ trợ serialization; nếu đổi sang third-party cache hoặc điều chỉnh `readOnly`, semantics về object copy và thread safety cũng thay đổi, cần xác nhận theo implementation cụ thể.

### Tại sao project production cần thận trọng khi dùng cache cấp hai của MyBatis?

- Cache được tổ chức theo `namespace`; khi Mapper khác sửa cùng một table, cache của namespace khác không tự biết.
- Data có thể bị sửa bởi application khác, script hoặc SQL trực tiếp, MyBatis không thể chủ động nhận biết.
- Local cache cấp hai chỉ tồn tại trong instance application hiện tại, không tự đồng bộ giữa nhiều instance.
- Join query phụ thuộc vào nhiều table, thay đổi ở bất kỳ table nào cũng có thể khiến result hết hạn.
- Khi cache object lớn hoặc hit rate thấp, nó chiếm memory nhưng hiệu quả hạn chế.

Khi cần share giữa các instance và có invalidation strategy rõ ràng, thông thường nên dùng cache ở business layer với Redis, Caffeine và các giải pháp khác, đồng thời coi database là data source có thẩm quyền. Cache cấp hai của MyBatis phù hợp hơn với query có ít thay đổi, read nhiều write ít và dependency rõ ràng.

### Cache cấp một có thể gây dirty read không?

Sau query đầu tiên trong cùng một `SqlSession`, nếu database bị transaction hoặc application khác update, lần query giống nhau tiếp theo của Session này có thể vẫn trả về object cũ trong cache cấp một. Đây không phải dirty read theo nghĩa database isolation level, nhưng thể hiện dưới dạng application nhìn thấy data cũ.

Khi cần lấy data mới nhất từ bên ngoài, có thể gọi `clearCache()`, rút ngắn Session lifecycle, đặt `localCacheScope` thành `STATEMENT`, hoặc để query dùng transaction/Session mới. Không nên tùy tiện clear cache trong một transaction rồi kỳ vọng database chắc chắn đã nhìn thấy value mới từ transaction khác; kết quả cuối cùng vẫn chịu ảnh hưởng của database isolation level.

## Executor, batch processing và plugin

### MyBatis có những Executor nào?

| ExecutorType | Behavior                                                                              |
| ------------ | ------------------------------------------------------------------------------------- |
| `SIMPLE`     | Tạo `PreparedStatement` mới cho mỗi lần execute, là type mặc định                     |
| `REUSE`      | Reuse `PreparedStatement` tương ứng với SQL trong `SqlSession` hiện tại               |
| `BATCH`      | Reuse Statement cho update và gọi JDBC Batch, flush batch trước query hoặc khi commit |

Scope của cả ba Executor đều đi theo `SqlSession`. `REUSE` reuse Statement, không đồng nghĩa với reuse query result; việc reuse query result do cache xử lý.

Có thể chọn Executor thông qua global `defaultExecutorType`, `openSession(ExecutorType)` hoặc constructor parameter của `SqlSessionTemplate`. Trong cùng một Spring transaction không nên đổi ExecutorType; khi thực sự cần type khác, nên dùng transaction độc lập hoặc thực thi ngoài transaction.

### ⭐️MyBatis thực thi batch processing như thế nào?

Có thể dùng `ExecutorType.BATCH`, lặp lại việc gọi `insert`, `update` hoặc `delete` của Mapper, cuối cùng thực thi `flushStatements()` hoặc commit transaction. `BatchExecutor` tổ chức batch theo Statement và SQL, sau đó gọi JDBC `addBatch()`, `executeBatch()`.

Cần chú ý khi batch processing:

- Không nên tích lũy vô hạn data trong một lần, nên `flushStatements()` theo batch cố định và clear session cache.
- Query ở giữa sẽ trigger flush các batch đã có.
- Return value của một Mapper call không nhất thiết là số row bị ảnh hưởng cuối cùng; cần lấy batch result sau khi flush.
- Khi xảy ra `BatchUpdateException`, một phần statement phía trước có thể đã được database execute; cần kết hợp transaction rollback và driver return result để phán đoán.
- Batch insert SQL, JDBC Batch và Bulk Load do database cung cấp là các giải pháp khác nhau, performance và khả năng generated key cũng khác nhau.

Trong MyBatis-Spring không được tự commit hoặc close `SqlSessionTemplate`, nên để Spring transaction manager commit và rollback.

### Batch insert có thể backfill primary key không?

Nếu database và JDBC driver hỗ trợ, có thể dùng `useGeneratedKeys="true"` và `keyProperty` để backfill auto-increment primary key. Khi parameter là object list, driver cần trả về generated keys của từng row chính xác thì MyBatis mới có thể lần lượt ghi ngược vào object.

```xml
<insert id="batchInsert"
        useGeneratedKeys="true"
        keyProperty="id">
  INSERT INTO users (name, status)
  VALUES
  <foreach collection="list" item="item" separator=",">
    (#{item.name}, #{item.status})
  </foreach>
</insert>
```

Mức độ hỗ trợ khác nhau khá lớn giữa database, driver version và cách viết batch SQL, nên cần dùng integration test để kiểm tra số lượng và thứ tự primary key. Khi không thể dựa vào JDBC generated keys, có thể generate UUID, Snowflake ID ở application side, hoặc dùng `<selectKey>` để lấy primary key theo khả năng của database.

### `useGeneratedKeys` và `<selectKey>` khác nhau như thế nào?

- `useGeneratedKeys` dùng JDBC `getGeneratedKeys()`, thường dùng cho database auto-increment primary key.
- `<selectKey>` thực thi thêm một query, có thể cấu hình chạy trước hoặc sau insert, phù hợp với sequence, database function đặc thù hoặc legacy database.

`keyProperty` chỉ định Java property nào nhận giá trị backfill; khi cần, dùng `keyColumn` để chỉ rõ database column. Batch insert, composite primary key và multi-data-source scenario cần được kiểm tra riêng, không thể suy ra behavior batch giống với single insert chỉ vì single insert đã thành công.

### ⭐️MyBatis pagination như thế nào? Nguyên lý của pagination plugin là gì?

MyBatis cung cấp `RowBounds`, nhưng không tự động thêm `LIMIT` vào SQL. Theo mặc định, nó vẫn execute SQL gốc, sau đó result set processing logic bỏ qua `offset` và giới hạn số lượng result trả về. Khi data volume lớn hoặc offset rất sâu, cách này có thể đọc rất nhiều record không cần thiết.

Production query thường dùng hai cách sau:

- Viết rõ physical pagination được database hỗ trợ trong SQL.
- Dùng pagination plugin như PageHelper, rewrite SQL theo database dialect trước khi execute và generate Count query khi cần.

Pagination plugin thông qua MyBatis plugin mechanism để intercept các object như `Executor` hoặc `StatementHandler`, đọc `MappedStatement`, `BoundSql` và pagination parameter, sau đó tạo pagination SQL mới. Plugin vẫn phải xử lý dialect, parameter order, Count SQL, cleanup thread context và multi-data-source.

Ngay cả khi thêm `LIMIT offset, size`, deep pagination vẫn có thể scan và discard rất nhiều record. Có thể dùng cursor pagination dựa trên stable sort key, ví dụ `WHERE id > ? ORDER BY id LIMIT ?`.

### Nguyên lý của MyBatis plugin là gì? Viết như thế nào?

MyBatis plugin có thể intercept method cụ thể của bốn loại component sau:

- `Executor`
- `StatementHandler`
- `ParameterHandler`
- `ResultSetHandler`

Plugin implement `Interceptor`, dùng `@Intercepts` và `@Signature` để khai báo target interface, method và parameter type. Khi MyBatis tạo component, nó gọi `pluginAll()`; target khớp signature sẽ được wrap bằng JDK dynamic proxy, khi gọi target method sẽ đi vào `intercept()`.

```java
@Intercepts({
    @Signature(
        type = Executor.class,
        method = "update",
        args = {MappedStatement.class, Object.class}
    )
})
public class AuditInterceptor implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // Chỉ xử lý phần cần thiết và bảo đảm cuối cùng gọi method gốc
        return invocation.proceed();
    }
}
```

Plugin đi vào main flow của mọi SQL match, code nên nhẹ và tránh sửa `MappedStatement` dùng chung. Thứ tự thực thi của nhiều plugin chịu ảnh hưởng bởi configuration order và proxy nesting; khi pagination, audit và data permission plugin cùng rewrite SQL, cần làm integration test kết hợp.

## Transaction, performance và vấn đề engineering

### Transaction hoạt động như thế nào trong MyBatis-Spring?

MyBatis-Spring dùng `SqlSessionTemplate` để bind `SqlSession` vào Spring transaction hiện tại. Mapper call trong cùng transaction và cùng `SqlSessionFactory` sẽ dùng chung Session và database connection; khi method kết thúc bình thường thì commit, khi ném exception phù hợp với rollback rule thì rollback.

Các điểm thường cần chú ý:

- `@Transactional` phải được gọi thông qua Spring proxy; self-invocation có thể bypass transaction interception.
- Transaction manager và `SqlSessionFactory` nên dùng cùng một `DataSource`.
- Không tự gọi `commit()`, `rollback()` hoặc `close()` trên `SqlSession` do Spring quản lý.
- Async thread không tự động inherit transaction của thread gốc.
- Sau khi catch exception mà không throw tiếp, có thể khiến Spring hiểu nhầm là method hoàn tất bình thường.

### Tại sao Mapper call thành công nhưng data lại không commit?

Các nguyên nhân thường gặp gồm:

- Khi dùng thủ công `openSession()`, mặc định không phải auto-commit và cũng không gọi `commit()`.
- Spring transaction cuối cùng rollback, hoặc transaction method ném runtime exception.
- `@Transactional` không có hiệu lực do self-invocation, object không do Spring quản lý hoặc method visibility.
- Write và query dùng khác data source, read replica chưa sync xong.
- Sau khi dùng `BatchExecutor` chưa flush batch hoặc commit transaction.

Khi kiểm tra, cần đồng thời quan sát transaction log, data source, trạng thái auto-commit của connection và exception cuối cùng; không thể chỉ xem Mapper method có throw error hay không.

### Dùng MyBatis để xử lý result set lớn như thế nào?

Trả về `List` một lần sẽ đưa toàn bộ result và object đã mapping vào memory. Khi data volume lớn có thể cân nhắc:

- Dùng `Cursor<T>` để iterate từng record.
- Dùng `ResultHandler` để xử lý từng row trong callback.
- Query theo batch dựa trên stable sort key.
- Khi export data, vừa đọc vừa ghi và giới hạn buffer size.

`Cursor` phụ thuộc vào `SqlSession`, connection và result set chưa bị close; quá trình consume phải nằm trong lifecycle của chúng. `fetchSize` chỉ là gợi ý cho JDBC driver, các database khác nhau có thể còn yêu cầu cursor hoặc connection configuration đặc thù. Trong thời gian streaming query, connection bị giữ lâu hơn, cần thiết lập timeout, rate limit và tránh trigger N+1 query trong loop.

### Kiểm tra vấn đề SQL performance của MyBatis như thế nào?

1. Lấy SQL cuối cùng và parameter thực tế, xác nhận dynamic condition đúng với dự kiến.
2. Xem execution plan, số row scan, số row trả về, sort và temporary table trong database.
3. Kiểm tra index, implicit type conversion, function calculation, pattern matching và deep pagination.
4. Kiểm tra N+1, việc load quá nhiều column trong một lần và batch operation bị suy biến thành loop từng statement.
5. Phân biệt thời gian chờ connection pool, database execution, result set transfer và Java object mapping.
6. Tiếp tục kiểm tra plugin, TypeHandler, log và cache có tạo thêm overhead hay không.

MyBatis chỉ là một phần của SQL execution chain. “Mapper method chậm” không nhất thiết có nghĩa SQL chậm; cũng có thể do connection pool cạn, result set quá lớn hoặc object mapping dùng nhiều CPU.

### Dùng MyBatis để tránh SQL injection như thế nào?

- Value parameter mặc định dùng `#{}`, không nối user input vào SQL.
- Table name, column name và sort direction được mapping qua server-side enum thành SQL fragment cố định.
- Dynamic SQL chỉ điều khiển việc output trusted fragment, không thực thi expression do user cung cấp.
- Database account tuân theo least privilege để giảm ảnh hưởng sau khi injection thành công.
- Việc nối string trong plugin và SQL Provider cũng cần được security review tương tự.

Parameterized query chỉ bảo vệ vị trí có parameter. `ORDER BY ${sort}`, `${tableName}` và SQL được nối từ annotation Provider vẫn cần whitelist.

### `id` trong các Mapper XML khác nhau có thể trùng không?

Có, với điều kiện `namespace` khác nhau. Full ID của `MappedStatement` là `namespace + id`, ví dụ:

```text
com.example.UserMapper.selectById
com.example.OrderMapper.selectById
```

Trong cùng một `namespace` không thể register hai full ID giống nhau. Khi Mapper XML dùng cùng interface, `namespace` thường ghi fully qualified name của Mapper interface.

### Khi sử dụng MyBatis còn có những hiểu lầm thường gặp nào?

- Cho rằng dùng MyBatis thì sẽ không có SQL injection, bỏ qua việc nối string trong `${}`, Provider và plugin.
- Coi Mapper method overload là có thể mapping nhiều SQL cùng name.
- Cho rằng `RowBounds` chắc chắn là physical pagination ở database.
- Coi cache cấp hai là business cache nhất quán giữa các application.
- Lưu native `SqlSession` trong singleton object.
- Dùng enum ordinal để lưu vào database, sau đó thay đổi thứ tự enum trong version tiếp theo.
- Truy cập lazy-loaded property trong loop, gây N+1.
- Batch write chỉ đếm số lần gọi method mà không xác nhận JDBC driver có thực sự execute Batch hay không.
- Dùng `LIMIT 1` để che giấu vấn đề data đáng ra phải được đảm bảo bằng unique constraint.

## Tài liệu tham khảo

- [Tài liệu chính thức MyBatis 3: Configuration](https://mybatis.org/mybatis-3/configuration.html)
- [Tài liệu chính thức MyBatis 3: Mapper XML Files](https://mybatis.org/mybatis-3/sqlmap-xml.html)
- [Tài liệu chính thức MyBatis 3: Dynamic SQL](https://mybatis.org/mybatis-3/dynamic-sql.html)
- [Tài liệu chính thức MyBatis 3: Java API](https://mybatis.org/mybatis-3/java-api.html)
- [Tài liệu chính thức MyBatis-Spring: Using an SqlSession](https://mybatis.org/spring/sqlsession.html)
- [Thảo luận về method overload của MyBatis Mapper](https://github.com/Snailclimb/JavaGuide/issues/1122)

## Bài viết đề xuất

- [Phân tích toàn diện 9 design pattern trong MyBatis với 20.000 chữ](https://juejin.cn/post/7273516671574687759)
- [Tự triển khai encryption/decryption plugin cho MyBatis từ đầu](https://mp.weixin.qq.com/s/WUEAdFDwZsZ4EKO8ix0ijg)
- [Hướng dẫn sử dụng MyBatis đầy đủ nhất](https://juejin.cn/post/7051910683264286750)
- [MyBatis cũng gặp vấn đề concurrency](https://juejin.cn/post/7264921613551730722)

<!-- @include: @article-footer.snippet.md -->
